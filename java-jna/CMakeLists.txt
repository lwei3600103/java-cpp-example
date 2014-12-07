# Make a shared library of cpplib, wrapped as C
# (JNA doesn't really do C++)
INCLUDE_DIRECTORIES("${CPPLIB_INCLUDE_DIR}")
ADD_LIBRARY(cpplib_shared SHARED c/c_wrapper.cpp)
TARGET_LINK_LIBRARIES(cpplib_shared cpplib)

# Require Java
FIND_PACKAGE(Java COMPONENTS Development)
INCLUDE(UseJava)
SET(CMAKE_JAVA_COMPILE_FLAGS "-source" "1.6" "-target" "1.6")

SET(JAVA_SOURCE_FILES
    edu/nyu/cpptest/app/Main.java
    edu/nyu/cpptest/cpplib/MyClass.java)

# JNA jar
SET(JNA_JAR "${CMAKE_CURRENT_SOURCE_DIR}/jna-4.1.0.jar"
    CACHE STRING "Location of jna.jar")

# Build Java classes
FILE(MAKE_DIRECTORY "${CMAKE_CURRENT_BINARY_DIR}/java/bin")
SET(class_files)
FOREACH(_java_file ${JAVA_SOURCE_FILES})
    # _java_file: relative file name
    # _class_file: relative class name
    STRING(REGEX REPLACE "\\.java$"
           ".class" _class_file
           "${_java_file}")
    ADD_CUSTOM_COMMAND(
        OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/java/bin/${_class_file}"
        COMMAND
            ${Java_JAVAC_EXECUTABLE}
            ${CMAKE_JAVA_COMPILE_FLAGS}
            -classpath "${JNA_JAR}"
            -sourcepath "${CMAKE_CURRENT_SOURCE_DIR}/java/src"
            -d "${CMAKE_CURRENT_BINARY_DIR}/java/bin"
            "${CMAKE_CURRENT_SOURCE_DIR}/java/src/${_java_file}")
    LIST(APPEND class_files "${CMAKE_CURRENT_BINARY_DIR}/java/bin/${_class_file}")
ENDFOREACH()
ADD_CUSTOM_TARGET(JavaJNAClasses ALL DEPENDS ${class_files})

# TODO: testing
# java -cp java-jna/java/bin:../java-jna/jna-4.1.0.jar:java-jna edu.nyu.cpptest.app.Main