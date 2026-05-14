set(CORAL_ROOT "${CMAKE_CURRENT_SOURCE_DIR}")

# ---- Coral.Managed (C# → dotnet build) ------------------------------------

find_program(DOTNET dotnet REQUIRED)

set(CORAL_BUILD_DIR "${CMAKE_BINARY_DIR}/Razor/vendor/Coral/cmake")

file(GLOB CORAL_MANAGED_SRC CONFIGURE_DEPENDS
    "${CORAL_ROOT}/Coral.Managed/Source/*.cs"
)

add_custom_command(
    OUTPUT
        "${CORAL_BUILD_DIR}/Coral.Managed.dll"
        "${CORAL_BUILD_DIR}/Coral.Managed.runtimeconfig.json"
    DEPENDS ${CORAL_MANAGED_SRC}
        "${CORAL_ROOT}/Coral.Managed/Coral.Managed.csproj"
    WORKING_DIRECTORY "${CORAL_ROOT}/Coral.Managed"
    COMMENT "Building Coral managed code"
    VERBATIM
    COMMAND ${DOTNET} build "${CORAL_ROOT}/Coral.Managed/Coral.Managed.csproj"
        -c $<CONFIG>
        -o "${CORAL_BUILD_DIR}"
        --nologo
)
add_custom_target(CoralManaged
    DEPENDS
        "${CORAL_BUILD_DIR}/Coral.Managed.dll"
        "${CORAL_BUILD_DIR}/Coral.Managed.runtimeconfig.json"
)

# ---- Coral.Native (C++ static library) ------------------------------------

file(GLOB_RECURSE CORAL_NATIVE_SRC
    "${CORAL_ROOT}/Coral.Native/Source/*.cpp"
    "${CORAL_ROOT}/Coral.Native/Source/*.hpp"
    "${CORAL_ROOT}/Coral.Native/Include/Coral/*.hpp"
)

add_library(Coral.Native STATIC ${CORAL_NATIVE_SRC})
add_dependencies(Coral.Native CoralManaged)

set_target_properties(Coral.Native PROPERTIES
    CXX_STANDARD 17
    CXX_STANDARD_REQUIRED ON
    POSITION_INDEPENDENT_CODE ON
)

target_precompile_headers(Coral.Native PRIVATE
    "${CORAL_ROOT}/Coral.Native/Source/CoralPCH.hpp"
)

target_include_directories(Coral.Native
    PUBLIC  "${CORAL_ROOT}/Coral.Native/Include"
    PRIVATE "${CORAL_ROOT}/Coral.Native/Source"
            "${CORAL_ROOT}/NetCore"
)

target_compile_definitions(Coral.Native PUBLIC
    $<$<PLATFORM_ID:Windows>:CORAL_WINDOWS>
    $<$<PLATFORM_ID:Linux>:CORAL_LINUX>
)
