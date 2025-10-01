# Create REST API with nodejs 24, express library
* Product catalog system

## List of APIs
1. Get product by id
id = number only

GET /product/:id

Success case 
GET /product/1
response code = 200
{
    "id": 1,
    "product_name": "Product 01",
    "price": 100.50
}

Product not found
GET /product/2
response code = 404
{
    "message": "Product id=2 not found in system"
}

Error
GET /product/3
response code = 500
{
    "message": "System Error"
}

## Project structure of API with NodeJs and Express library
* Flow of code
  * index.js -> routes/product.js -> service/product.js

```
root_project
- src/
  - index.js
  - routes/product.js
  - service/product.js
``` 

## Coding Best practices of NodeJS and Express
* Create Unit test for all endpoints and all cases (happy case, fail case, edge case, security case)
* Unit test with library => jest, supertest
* Validation all inputs
* Write clean code and human readable
