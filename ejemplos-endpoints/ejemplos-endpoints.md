# Ejemplo de guardado de tool http://localhost:8080/admin/tools/api
{
"name": "Precio de Bitcoin",
"code": "bitcoin_precio",
"authenticationType": "NONE",
"apiKeyLocation": "NONE",
"apiKeyName": null,
"apiKeyValue": null,
"enabled": true,
"healthy": true,
"description": "Obtiene el precio actual de Bitcoin en USD desde CoinGecko.",
"baseUrl": "https://api.coingecko.com",
"endpointPath": "/api/v3/simple/price",
"httpMethod": "GET",
"parameters": [
{
"name": "ids",
"type": "STRING",
"description": "ID de la criptomoneda",
"required": true,
"defaultValue": "bitcoin"
},
{
"name": "vs_currencies",
"type": "STRING",
"description": "Moneda de referencia para el precio",
"required": true,
"defaultValue": "usd"
}
]
}

# Ejemplo de post a tool http://localhost:8080/mcp/tools/call
{
"jsonrpc": "2.0",
"id": 1,
"method": "tools/call",
"params": {
"name": "clima",
"arguments": {
"q": "Madrid",
"key": null
}
}
}