Portainer: https://localhost:9443

Keycloak: http://localhost:8081

Create Token from Keycloak
curl -s -X POST \
  'http://localhost:8081/realms/dsa-lab/protocol/openid-connect/token' \
  -d 'client_id=rest-client' \
  -d 'grant_type=password' \
  -d 'username=tvph1996' \
  -d 'password=H.g.t.124' \
  | jq -r .access_token > token.jwt
