```cmake
get_cmake_property(_variableNames VARIABLES)

list(SORT _variableNames)

foreach(_variableName ${_variableNames})
    message("${_variableName}=${${_variableName}}")
    file(APPEND "var.log" "${_variableName} = ${${_variableName}}\n")
endforeach()
```
