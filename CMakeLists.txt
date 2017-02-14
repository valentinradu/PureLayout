cmake_minimum_required (VERSION 3.7)

file(GLOB_RECURSE LIBRARY_OBJECTS PureLayout/PureLayout/*.m)
file(GLOB_RECURSE LIBRARY_HEADERS PureLayout/PureLayout/*.h)

add_library(
    ${FRAMEWORK_NAME} SHARED
    ${LIBRARY_HEADERS}
    ${LIBRARY_OBJECTS}
)

# Locate system libraries on iOS
find_library(UIKIT UIKit)
find_library(FOUNDATION Foundation)

# link the frameworks located above
target_link_libraries(${FRAMEWORK_NAME} ${UIKIT})
target_link_libraries(${FRAMEWORK_NAME} ${FOUNDATION})

set_target_properties(${FRAMEWORK_NAME} PROPERTIES
    FRAMEWORK TRUE
    FRAMEWORK_VERSION A
    MACOSX_FRAMEWORK_IDENTIFIER ${FRAMEWORK_BUNDLE_IDENTIFIER}
    MACOSX_FRAMEWORK_INFO_PLIST ${CMAKE_CURRENT_LIST_DIR}/framework.plist.in
    # "current version" in semantic format in Mach-O binary file
    VERSION 1.0.0
    # "compatibility version" in semantic format in Mach-O binary file
    SONAME ${FRAMEWORK_NAME}
    SOVERSION 1.0.0
    PUBLIC_HEADER "${LIBRARY_HEADERS}"
    XCODE_ATTRIBUTE_CODE_SIGN_IDENTITY ${CODESIGNIDENTITY}
    XCODE_ATTRIBUTE_DEVELOPMENT_TEAM ${DEVELOPMENT_TEAM_ID}
)

add_custom_command(
    TARGET ${FRAMEWORK_NAME}
    POST_BUILD
    COMMAND /bin/bash -c "${CMAKE_CURRENT_LIST_DIR}/install_name.sh $<TARGET_FILE:${FRAMEWORK_NAME}>"
)

add_custom_command(
    TARGET ${FRAMEWORK_NAME}
    POST_BUILD
    COMMAND install_name_tool -id \"@rpath/${FRAMEWORK_NAME}.framework/${FRAMEWORK_NAME}\"
    $<TARGET_FILE:${FRAMEWORK_NAME}>
)

