# JSON Maker

[![Build Status](https://travis-ci.org/rafagafe/json-maker.svg?branch=master)](https://travis-ci.org/rafagafe/json-maker)

JSON Maker is a C library used to code JSON objects in null-terminated strings.

Surely the most effective method to create simple JSON objects is to use sprintf. But when you need to reuse code, nest objects or include arrays you can fall into the formatted-strings hell.

* Backslash escapes are automatically added. Only in the fields of type string.
* By means of compilation options, the use of print can be avoided. This is very useful in embedded systems with memory constraint.

If you need a JSON parser please visit: https://github.com/rafagafe/tiny-json

# Philosophy

To form JSON objects in strings of characters are invoked sequences of functions that concatenate sub-strings. Each substring includes a field of the JSON object, key-value. To add fields of the type object or array you need to invoke two functions, one to open and another to close.

```C

struct weather {
    int temp;
    int hum;
};

/** Convert a weather structure in a JSON string.
  * @param dest Destination memory block.
  * @param src Source structure.
  * @return The length of the null-terminated string in dest. */
int weather_to_json( char* dest, struct weather const* src, size_t* remLen ) {
    char* p = dest;                               // p always points to the null character
    p = json_objOpen( p, NULL, remLen );          // --> {\0
    p = json_int( p, "temp", src->temp, remLen ); // --> {"temp":22,\0
    p = json_int( p, "hum", src->hum, remLen );   // --> {"temp":22,"hum":45,\0
    p = json_objClose( p, remLen );               // --> {"temp":22,"hum":45},\0
    p = json_end( p, remLen );                    // --> {"temp":22,"hum":45}\0
    return p - dest;       
}
    
```

The complexity of these sequences of concatenations is kept in O(n) thanks to the fluent interface of JSON Maker.

It is very easy to extend the library by creating methods to convert C structures into JSON fields of object type. As with the arrays.

```C

struct weather {
    int temp;
    int hum;
};

/* Add a time object property in a JSON string.
  "name":{"temp":-5,"hum":48}, */
char* json_weather( char* dest, char const* name, struct weather const* weather, size_t* remLen ) {
    // dest always points to the null character
    dest = json_objOpen( dest, name, remLen );              // --> "name":{\0
    dest = json_int( dest, "temp", weather->temp, remLen ); // --> "name":{"temp":22,\0
    dest = json_int( dest, "hum", weather->hum, remLen );   // --> "name":{"temp":22,"hum":45,\0
    dest = json_objClose( dest, remLen );                   // --> "name":{"temp":22,"hum":45},\0
    return dest;
}

struct time {
    int hour;
    int minute;
};

/* Add a time object property in a JSON string.
  "name":{"hour":18,"minute":32}, */
char* json_time( char* dest, char const* name, struct time const* time, size_t* remLen ) {
    dest = json_objOpen( dest, name, remLen );
    dest = json_int( dest, "hour",   time->hour, remLen   );
    dest = json_int( dest, "minute", time->minute, remLen );
    dest = json_objClose( dest, remLen );
    return dest;
}

struct measure {
    struct weather weather;
    struct time time;
};

/** Convert a weather structure in a JSON string.
  * {"weather":{"temp":-5,"hum":48},"time":{"hour":18,"minute":32}}
  * @param dest Destination memory block.
  * @param src Source structure.
  * @return The length of the null-terminated string in dest. */
int measure_to_json( char* dest, struct measure const* measure, size_t* remLen ) {
    char* p = json_objOpen( dest, NULL, remLen );
    p = json_weather( p, "weather", &measure->weather, remLen );
    p = json_time( p, "time", &measure->time, remLen );
    p = json_objClose( p, remLen );
    p = json_end( p, remLen );
    return p - dest;
}

```

To see more nested JSON objects and arrays please read example.c.

#Building and Testing

JSON Maker is built as a static library.
JSON Maker relies on CMake and CTest for building and testing (see [https://cmake.org/](https://cmake.org/) for more information).

The library is found under `{project_root}/build/lib/static`, the executables under `{project_root}/build/bin`.

Two configurations are supported, `Debug` and `Release`. You can use option `CMAKE_BUILD_TYPE` to change from the default (`Debug`).
The following examples assume `Debug` builds, except for install which uses `Release`.
## Create the build folder 

```shell
mkdir build
cd build
```

##Building the static library

```shell
cmake --configure ..
cmake --build .
```

##Installing the static library
if you are not super-user or don't want to use sudo you can install the library in your home's `opt` folder.
```shell
cmake --configure -DCMAKE_BUILD_TYPE=Release..
cmake --build .
```
As sudo/root (will use default `/usr/local/` on Unix)
```
cmake --install .
```
As a regular user 
```
cmake --install . --prefix $HOME/opt
```

##Building the sample application

```shell
cmake --configure -DBUILD_SAMPLES=ON ..
cmake --build .
```

##Runing the tests

```shell
cmake --configure ..
cmake --build .
cd tests
ctest .
```