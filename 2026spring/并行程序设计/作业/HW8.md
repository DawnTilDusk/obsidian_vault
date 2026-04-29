```cpp title:realarray_test.cpp unwrap:true
#include <gtest/gtest.h>
#include "realarray.h"

using ModuleBase::realArray;

TEST(RealArray, DefaultConstructor) {
    realArray a;
    EXPECT_EQ(a.getDim(), 3);
    EXPECT_EQ(a.getSize(), 1);
    EXPECT_EQ(a.getBound1(), 1);
    EXPECT_EQ(a.getBound2(), 1);
    EXPECT_EQ(a.getBound3(), 1);
}

TEST(RealArray, Constructor_3D_Array) {
    realArray a(2, 3, 2);
    EXPECT_EQ(a.getDim(), 3);
    EXPECT_EQ(a.getBound1(), 2);
    EXPECT_EQ(a.getBound2(), 3);
    EXPECT_EQ(a.getBound3(), 2);
    EXPECT_EQ(a.getSize(), 2 * 3 * 2);
}

TEST(RealArray, Constructor_4D_Array) {
    realArray a(2, 2, 2, 2);
    EXPECT_EQ(a.getDim(), 4);
    EXPECT_EQ(a.getBound1(), 2);
    EXPECT_EQ(a.getBound2(), 2);
    EXPECT_EQ(a.getBound3(), 2);
    EXPECT_EQ(a.getBound4(), 2);
    EXPECT_EQ(a.getSize(), 2 * 2 * 2 * 2);
}

TEST(RealArray, Create_for_3D_Array) {
    realArray a;
    a.create(3, 2, 2);
    EXPECT_EQ(a.getDim(), 3);
    EXPECT_EQ(a.getBound1(), 3);
    EXPECT_EQ(a.getBound2(), 2);
    EXPECT_EQ(a.getBound3(), 2);
    EXPECT_EQ(a.getSize(), 3 * 2 * 2);
}

TEST(RealArray, Create_for_4D_Array) {
    realArray a;
    a.create(2, 2, 2, 2);
    EXPECT_EQ(a.getDim(), 4);
    EXPECT_EQ(a.getBound1(), 2);
    EXPECT_EQ(a.getBound2(), 2);
    EXPECT_EQ(a.getBound3(), 2);
    EXPECT_EQ(a.getBound4(), 2);
    EXPECT_EQ(a.getSize(), 2 * 2 * 2 * 2);
}

TEST(RealArray, Element_Access_for_3D_Array) {
    realArray a(2, 2, 2);
    for (int i = 0; i < 2; i++)
        for (int j = 0; j < 2; j++)
            for (int k = 0; k < 2; k++)
                a(i, j, k) = 100 * i + 10 * j + k;

    for (int i = 0; i < 2; i++)
        for (int j = 0; j < 2; j++)
            for (int k = 0; k < 2; k++)
                EXPECT_DOUBLE_EQ(a(i, j, k), 100 * i + 10 * j + k);
}

TEST(RealArray, Scalar_Assignment) {
    realArray a(2, 2, 2);
    a = 5.5;
    for(int i = 0; i < 2; i++)
        for(int j = 0; j < 2; j++)
            for(int k = 0; k < 2; k++)
                EXPECT_DOUBLE_EQ(a(i, j, k), 5.5);
}

TEST(RealArray, Zero_Out) {
    realArray a(2, 2, 2);
    a.zero_out();
    for(int i = 0; i < 2; i++)
        for(int j = 0; j < 2; j++)
            for(int k = 0; k < 2; k++)
                EXPECT_DOUBLE_EQ(a(i, j, k), 0.0);
}

TEST(RealArray, Copy) {
    realArray a(2, 2, 2);
    for (int i = 0; i < 2; i++)
        for (int j = 0; j < 2; j++)
            for (int k = 0; k < 2; k++)
                a(i, j, k) = 400 * i + 20 * j + k;
    
    realArray b = a;
	
    for (int i = 0; i < 2; i++)
        for (int j = 0; j < 2; j++)
            for (int k = 0; k < 2; k++)
                EXPECT_DOUBLE_EQ(b(i, j, k), 400 * i + 20 * j + k);
}

TEST(RealArray, Operations_for_4D_Array) {
    realArray a(2, 2, 2, 2);
    for (int i = 0; i < 2; i++)
        for (int j = 0; j < 2; j++)
            for (int k = 0; k < 2; k++)
                for (int l = 0; l < 2; l++)
                    a(i, j, k, l) = 100 * i + 10 * j + 20 * k + 30 * l;
        
    for (int i = 0; i < 2; i++)
        for (int j = 0; j < 2; j++)
            for (int k = 0; k < 2; k++)
                for (int l = 0; l < 2; l++)
                    EXPECT_DOUBLE_EQ(a(i, j, k, l), 100 * i + 10 * j + 20 * k + 30 * l);
    
    a.zero_out();
	
    for (int i=0; i < 2; i++)
        for (int j=0; j < 2; j++)
            for (int k=0; k < 2; k++)
                for (int l=0; l < 2; l++)
                    EXPECT_DOUBLE_EQ(a(i, j, k, l), 0.0);
}
```

```sh title:compile.sh
#!/bin/bash

echo "[INFO] Checking if gtest is compiled..."
GTEST_MAIN_LIB="/usr/src/googletest/build/lib/libgtest_main.a"
if [ ! -f $GTEST_MAIN_LIB ]; then
    echo "[ERROR] gtest is not compiled."
    echo "[INFO] Compliling gtest..."
    cd /usr/src/googletest
    cmake -S . -B build
    cmake --build build -j
    exit 1
else
    echo "[INFO] gtest is compiled!"
fi


# Script to compile realArray test program
echo "[INFO] Compiling realArray test program..."

# Compiler settings
CXX=g++
CXXFLAGS="-std=c++11 -Wall -Wextra -I../ -I/usr/src/googletest/googletest/include"

# Source files
SRC_FILES="realarray_test.cpp ../realarray.cpp"

# Output executable
OUTPUT="realarray_test"

#gtest
GTEST_LIB="/usr/src/googletest/build/lib/libgtest.a"
GTEST_MAIN_LIB="/usr/src/googletest/build/lib/libgtest_main.a"

# Compile command
$CXX $CXXFLAGS $SRC_FILES $GTEST_LIB $GTEST_MAIN_LIB -pthread -o $OUTPUT -lgtest -lgtest_main

if [ $? -eq 0 ]; then
    echo "[INFO] Compilation successful!"
    echo "[INFO] Run the test program with: ./$OUTPUT"
    echo "[INFO] running test..."
    ./$OUTPUT
else
    echo "[ERROR] Compilation failed!"
    exit 1
fi
```

![[Pasted image 20260408113220.png]]