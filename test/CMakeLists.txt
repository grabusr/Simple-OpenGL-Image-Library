set(TEST_NAME SoilTests)

project(${TEST_NAME})

aux_source_directory(. TEST_SRC)

add_executable(${TEST_NAME} TEST_SRC)
target_link_libraries(${TEST_NAME} SoilLib)
install(TARGETS SoilLib ${TEST_NAME} RUNTIME DESTINATION bin)