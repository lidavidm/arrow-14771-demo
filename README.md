# Demo for ARROW-14471

https://issues.apache.org/jira/browse/ARROW-14771

This project links a trivial executable with Protobuf-generated code that attempts to depend on Flight.proto.

Without a patch:

```
$ mold -run ninja
[4/4] Linking CXX executable demo
FAILED: demo
: && /home/lidavidm/miniconda3/envs/dev/bin/c++   CMakeFiles/demo.dir/src/main.cc.o CMakeFiles/demo.dir/test.pb.cc.o -o demo  -Wl,-rpath,/home/lidavidm/Code/upstream/arrow-14771/install/lib  /home/lidavidm/miniconda3/envs/dev/lib/libprotobuf.so  -pthread  /home/lidavidm/Code/upstream/arrow-14771/install/lib/libarrow.so.900.0.0  /home/lidavidm/Code/upstream/arrow-14771/install/lib/libarrow_flight.so.900.0.0  -ldl && :
mold: error: undefined symbol: CMakeFiles/demo.dir/test.pb.cc.o: descriptor_table_Flight_2eproto
collect2: error: ld returned 1 exit status
ninja: build stopped: subcommand failed.
```

If we try to generate and link the sources for Flight:

```
$ cmake [...] -DWITH_GENERATED_FLIGHT=ON
$ mold -run ninja
[5/5] Linking CXX executable demo
$ ./demo
[libprotobuf ERROR google/protobuf/descriptor_database.cc:641] File already exists in database: Flight.proto
[libprotobuf FATAL google/protobuf/descriptor.cc:2021] CHECK failed: GeneratedDatabase()->Add(encoded_file_descriptor, size):
terminate called after throwing an instance of 'google::protobuf::FatalException'
  what():  CHECK failed: GeneratedDatabase()->Add(encoded_file_descriptor, size):
fish: Job 1, './demo' terminated by signal SIGABRT (Abort)
```

With the fix:

```
$ mold -run ninja
[4/4] Linking CXX executable demo
$ ./demo
```
