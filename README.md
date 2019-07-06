APP_NAME=demo
SPACE_NAME=demo

MESH_DOMAIN=mesh.apps.pcfbeta.io

cf push --no-route

cf create-route ${SPACE_NAME} ${MESH_DOMAIN} -n ${APP_NAME}

APP_GUID=$(cf app ${APP_NAME} --guid)
ROUTE_GUID=$(cf curl /v2/routes?q=host:${APP_NAME} | jq -r .resources[0].metadata.guid)
cf curl /v3/route_mappings -X POST -d "{\"relationships\":{\"app\":{\"guid\":\"${APP_GUID}\"},\"route\":{\"guid\":\"${ROUTE_GUID}\"}},\"weight\":100}"


git checkout blue
cf push ${APP_NAME}-v2 --no-route

APP_GUID=$(cf app ${APP_NAME}-v2 --guid)
cf curl /v3/route_mappings -X POST -d "{\"relationships\":{\"app\":{\"guid\":\"${APP_GUID}\"},\"route\":{\"guid\":\"${ROUTE_GUID}\"}},\"weight\":10}"

cf curl /v3/route_mappings/$(cf curl /v3/apps/${APP_GUID}/route_mappings | jq -r '.resources[0].guid') -X PATCH -d '{"weight": 50}'
cf curl /v3/route_mappings/$(cf curl /v3/apps/${APP_GUID}/route_mappings | jq -r '.resources[0].guid') -X PATCH -d '{"weight": 90}'
cf curl /v3/route_mappings/$(cf curl /v3/apps/${APP_GUID}/route_mappings | jq -r '.resources[0].guid') -X PATCH -d '{"weight": 100}'