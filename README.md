[![Build Status](https://travis-ci.org/dart-league/dson.svg?branch=master)](https://travis-ci.org/dart-league/dson)

DSON is a dart library which converts Dart Objects into their JSON
representation.

This library was initially a fork from
[Dartson](https://github.com/eredo/dartson). Now it contains some
differences:

  - Dartson uses custom transformers to convert objects to JSON. This
    produce faster and smaller code after dart2Js. Instead DSON uses
    \[serializable\]() and \[built\_mirrors\]() libraries. This should
    produce code as fast and small as Dartson transformer.

  - DSON has the ability to serialize cyclical objects by mean of
    `depth` parameter, which allows users to specify how deep in the
    object graph they want to serialize.

  - DSON has the ability to exclude attributes for serialziation in two
    ways.

  - Using `@ignore` over every attribute. This make excluding attributes
    too global and hardcoded, so users can only specify one exclusion
    schema.

  - Using `exclude` map as parameter for `toJson` method. This is more
    flexible, since it allows to have many exclusion schemas for
    serialization.

  - DSON uses the annotation `@serializable` instead `@entity` which is
    used by Dartson.

# Comparison with other libraries

<https://github.com/drails-dart/dart-serialise>

# Tutorials

![DSON tutorials](http://img.youtube.com/vi/dZrCrCsw208/0.jpg)

# Configuration

1- Create a new dart project.

2- Add dependencies to `pubspec.yaml`

``` yaml
...
dependencies:
  ...
  dson: any # replace for latest version
  ...
```

3- Create/edit `bin/main.dart` or `web/main.dart` and add the code shown
in any of the samples below.

4- Run either `pub run build_runner build`, or `pub run build_runner
watch`, or `pub run build_runner serve` in the console

# Convert objects to JSON strings

To convert objects to JSON strings you only need to use the `toJson`
function, annotate the object with `@serializable` and pass the `object`
to the `toJson` function as parameter:

``` dart
library example.object_to_json; // this line is needed for the generator

import 'package:dson/dson.dart';

part 'object_to_json.g.dart'; // this line is needed for the generator

@serializable
class Person extends _$PersonSerializable {
  int id;
  String firstName;
  var lastName; //This is a dynamic attribute could be String, int, double, num, date or another type
  double height;
  DateTime dateOfBirth;

  @SerializedName("renamed")
  String otherName;

  @ignore
  String notVisible;

  // private members are never serialized
  String _private = "name";

  String get doGetter => _private;
}

void main() {
  _initMirrors();

  Person object = new Person()
    ..id = 1
    ..firstName = "Jhon"
    ..lastName = "Doe"
    ..height = 1.8
    ..dateOfBirth = new DateTime(1988, 4, 1, 6, 31)
    ..otherName = "Juan"
    ..notVisible = "hallo";

  String jsonString = toJson(object);
  print(jsonString);
  // will print: '{"id":1,"firstName":"Jhon","lastName":"Doe","height":1.8,"dateOfBirth":"1988-04-01T06:31:00.000","renamed":"Juan","doGetter":"name"}'
}
```

# Converting objects to Maps

To convert objects to Maps you only need to use the `toMap` function,
annotate the object with `@serializable` and pass the `object` to
`toMap` function as parameter:

``` dart
example/bin/object_to_map.dart
```

## Serializing Cyclical Objects

To serialize objects that contains Cyclical References it would be
needed to use the annotation `@cyclical`. If this annotation is present
and the `depth` variable is not set then the non-primitive objects are
not going to be parsed and only the id (or hashmap if the object does
not contains id) is going to be present. Let’s see next example:

``` dart
example/bin/serialize_cyclical.dart
```

as you can see employee has an address, and the address has an owner of
type Employee. If the property `id` is not present in the object then it
is going to take the `hashcode` value from the object as reference. And
finally, the `depth` parameter passed to serialize function tells
serializer how deep you want to go throw the reference. This help us not
only to avoid cyclical reference, but to determine what referenced
objects should be serialized.

The same applies for lists:

``` dart
example/bin/serialize_cyclical_list.dart
```

Without the annotation `@cyclical` the program is going to throw a stack
overflow error caused by the serializing of cyclical objects.

## Excluding attributes from being serialized

To exclude parameter from being serialized we have two options the first
option is using `@ignore` over the attribute to ignore. However this
approach is too global. What I want to say with this is that the
attribute is going to be ignored always.

Another way to exclude attributes is adding the parameter `exclude` to
`serialize` function. In this way we only exclude those attributes
during that serialization.

``` dart
example/bin/exclude_attributes.dart
```

# Convert JSON strings to objects

To convert JSON strings to objects you only need to use the `fromJson`
and `fromJsonList` functions and pass the `json` string to deserialize
and the `Type` of the object as parameters:

``` dart
example/bin/json_to_object.dart
```

## Converting `Maps` and `Lists<Map>` to dart objects

Frameworks like Angular.dart come with several HTTP services which
already transform the HTTP response to a map using `JSON.encode`. To use
those encoded Maps or Lists use `fromMap` function.

``` dart
example/bin/map_to_object.dart
```

# Extend serializable Objects

To extends objects that are going to be serializable you will need to
add the comment:

``` dart
// ignore: mixin_inherits_from_not_object
```

This is to advice the analyzer to ignore the error caused by inheriting
from an object that is not a mixin. For example:

``` dart
example/bin/extend_serializables.dart
```

# Serialize/Deserialize immutable objects

To make an immutable class to be able to serialize/deserialize you only
need to declare it with a constructor which only contains final
parameters. For example:

``` dart
example/bin/immutable_objects.dart
```

> Be sure the names of the fields and constructor parameters match. If
> they do not match, then the deserialized object will contain
> attributes with null value
