
The project uses `cmake-common`, and so upon cloning this repo
it is important you call this before running cmake:

```
git submodule update --init --recursive
```

```
cmake -B build
cmake --build build -- -j
```
