# **GOALS**

- Implement Traefik gateway as a single, secure entry point to force using of HTTPS for communication from the outside.
- Authentication and authorization are handled by Keycloak as an OAuth 2.0/OIDC provider, requiring valid JWT bearer tokens.
- A zero-trust internal network is created by updating to mutual TLS, forcing services to verify each other's identity.

&nbsp;

# **SYSTEM SETUP**

- Two new blocks added to the existing stack: Traefik & Keycloak
- Traefik acts as the gateway, managing all incoming HTTPS traffic and routing it to the internal services while hiding the rest of the system.
- Keycloak handles user login and gives temporary access tokens (JWTs) for authentication and authorization.
- Used in the lab is an own self-signed CA, which acts as the root of trust for other services, including a root private key (ca.key) for signing other certificates and a public certificate (ca.crt) for verification.
- Similarly, both rest-service and grpc-service also has their respective private key and public certificate.
- In additional, another pair of private key and public certificate is created for localhost, differs from using rest-service certificate as the lab guideline. This is to avoid making rest-service identity visible to the outside, in case the attacker steal localhost, he can’t call grpc-service -> MongoDB directly. Not only that, a separate hostname of a certificate is still better.
- mTLS is setup between rest-service and grpc-service. Both must prove their identity to each other before any communication is allowed, achieving a zero-trust security posture.

&nbsp;

# HOW TO RUN

Simply `docker compose up -d`

&nbsp;

# USEFUL COMMANDS

### **Portainer**

`https://localhost:9443`

### **Traefik**

`http://localhost:8080`

### **Keycloak**

`http://localhost:8081`

### **Create Token from Keycloak**

```
curl -s -X POST \

'http://localhost:8081/realms/dsa-lab/protocol/openid-connect/token' \

-d 'client_id=rest-client' \

-d 'grant_type=password' \

-d 'username=tvph1996' \

-d 'password=H.g.t.124' \

| jq -r .access_token > token.jwt
```

### **Add Item**

```
curl -X POST \

--cacert security/certs/ca.crt \

-H "Authorization: Bearer $(cat token.jwt)" \

-H "Content-Type: application/json" \

-d '{"id": 5, "name": "A Fifth Item"}' \

https://localhost/api/items
```

### **Get Item**

```
curl --cacert security/certs/ca.crt \

-H "Authorization: Bearer $(cat token.jwt)" \

https://localhost/api/items/?id=5
```

### **Create new key and crt**

```
openssl req -new -newkey rsa:2048 -nodes \

-keyout security/certs/rest-service.key -out security/certs/rest-service.csr \

-subj "/C=DE/O=FH-Anhalt/CN=rest-service"


openssl x509 -req -in security/certs/rest-service.csr \

-CA security/certs/ca.crt -CAkey security/certs/ca.key -CAcreateserial \

-out security/certs/rest-service.crt -days 180
```
