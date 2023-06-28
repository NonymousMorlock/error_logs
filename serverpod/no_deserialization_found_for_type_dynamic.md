## PROBLEM

I've tried serverpod out when it was fresh out, and back then I used generic types, it was just a simple example app, but now I'm implementing it in a project and I've run into some issues, so, I have this code to add a product to the database...I'm not sure if I'm doing the serialization properly, by using `Protocol()`, it's asking for a `SerializationManager`, and this is the protocol from src/generated, however it throws an error, I'm not sure what I'm doing wrong

PS: When I pass the product as a map to the `addProduct` endpoint, I don't give it any id, I intentionally exclude the id from it, as I didn't even add an id in the schema, but the Id is auto added, so I don't know if this contributes to the error in anyway

**SERVER**

```dart
  Future<Response> addProduct(
    Session session,
    Map<String, dynamic> body,
  ) async {
    try {
      final product = Product.fromJson(body, Protocol());
      await Product.insert(session, product);
      return Response(body: 'Product added', statusCode: 201);
    } catch (e) {
      return Response(body: e.toString(), statusCode: 500);
    }
  }
```

**CLIENT**
```dart
 final response = await _client.product.addProduct(furnitureMap);
      if (response.statusCode != 201) {
        throw ServerException(
          message: response.body,
          statusCode: response.statusCode,
        );
      }
```


**PRODUCT PROTOCOL**

```yaml
class: Product
table: product
serverOnly: true

fields:
  name: String
  sku: String?
  availability: String
  categories: List<String>
  images: List<String>
  weight: double
  weightUnit: String
  dimensions: Map<String, double>?
  price: double
  compareAtPrice: double?
  createdAt: int
  updatedAt: int
  availableAt: int

```

however, It throws whenever I hit the endpoint

```shell
flutter: Failed call: product.addProduct
flutter: ServerpodClientException: Internal server error. Call log id: 11, statusCode = 500
```

**STACKTRACE**

```shell
METHOD CALL: product.addProduct duration: 1ms numQueries: 0 authenticatedUser: null
FormatException: No deserialization found for type dynamic
#0      SerializationManager.deserialize (package:serverpod_serialization/src/serialization.dart:79:5)
#1      Protocol.deserialize (package:furniture_store_server_server/src/generated/protocol.dart:226:18)
#2      Protocol.deserialize.<anonymous closure> (package:furniture_store_server_server/src/generated/protocol.dart:221:44)
#3      MapBase.map (dart:collection/maps.dart:82:28)
#4      Protocol.deserialize (package:furniture_store_server_server/src/generated/protocol.dart:220:28)
#5      EndpointDispatch._formatArg (package:serverpod/src/server/endpoint_dispatch.dart:169:33)
#6      EndpointDispatch.handleUriCall (package:serverpod/src/server/endpoint_dispatch.dart:106:25)
<asynchronous suspension>
#7      Server._handleRequest (package:serverpod/src/server/server.dart:243:18)
<asynchronous suspension>
```

## SOLUTION

It couldn't handle the `Map<String, dynamic>` type that I was asking for in the `addProduct` endpoint signature
so I changed it to a `String` instead, and it worked, I'm not sure if this is the best way to do it, but it works

**SERVER**

```dart
  Future<Response> addProduct(
    Session session,
    String body,
  ) async {
    try {
      final product = Product.fromJson(jsonDecode(body), Protocol());
      await Product.insert(session, product);
      return Response(body: 'Product added', statusCode: 201);
    } catch (e) {
      return Response(body: e.toString(), statusCode: 500);
    }
  }
```
