First Step:

android {
...
externalNativeBuild {
    cmake {
        path "CMakeLists.txt"
    }
}
...
defaultConfig {
    externalNativeBuild {
        cmake {
            cppFlags "-std=c++14"
        }
    }
...
}
And the second step is to add the CMakeLists.txt file:

cmake_minimum_required(VERSION 3.4.1)

include_directories (
    ../../CPP/
)

add_library(
    native-lib
    SHARED
    src/main/cpp/native-lib.cpp
    ../../CPP/Core.h
    ../../CPP/Core.cpp
)

find_library(
    log-lib
    log
)

target_link_libraries(
    native-lib
    ${log-lib}
)
The CMake file is where you need to add the CPP files and header folders you will use on the project, on our example, we are adding the CPP folder and the Core.h/.cpp files. To know more about C/C++ configuration please read it.

Now the core code is part of our app it is time to create the bridge, to make the things more simple and organized we create a specific class named CoreWrapper to be our wrapper between JVM and CPP:

public class CoreWrapper {

    public native String concatenateMyStringWithCppString(String myString);

    static {
        System.loadLibrary("native-lib");
    }

}
Note this class has a native method and loads a native library named native-lib. This library is the one we create, in the end, the CPP code will become a shared object .so File embed in our APK, and the loadLibrary will load it. Finally, when you call the native method, the JVM will delegate the call to the loaded library.

Now the most strange part of Android integration is the JNI; We need a cpp file as follow, in our case "native-lib.cpp":

extern "C" {

JNIEXPORT jstring JNICALL Java_ademar_androidioscppexample_CoreWrapper_concatenateMyStringWithCppString(JNIEnv *env, jobject /* this */, jstring myString) {
    const char *utfString = env->GetStringUTFChars(myString, 0);
    const char *textFromCppCore = concatenateMyStringWithCppString(utfString);
    jstring javaString = env->NewStringUTF(textFromCppCore);
    return javaString;
}

}
