include_directories(${PROJECT_SOURCE_DIR})
include_directories(${PROJECT_BINARY_DIR})

enable_language(Fortran)

if (BUILD_SINGLE)
	list( APPEND OpenBLAS_Tests sblat1 sblat2 sblat3)
endif()
if (BUILD_DOUBLE)
	list (APPEND OpenBLAS_Tests dblat1 dblat2 dblat3)
endif()
if (BUILD_COMPLEX)
	list (APPEND OpenBLAS_Tests cblat1 cblat2 cblat3)
endif()
if (BUILD_COMPLEX16)
	list (APPEND OpenBLAS_Tests zblat1 zblat2 zblat3)
endif()

foreach(test_bin ${OpenBLAS_Tests})
	add_executable(${test_bin} ${test_bin}.f)
target_link_libraries(${test_bin} ${OpenBLAS_LIBNAME})
endforeach()

# $1 exec, $2 input, $3 output_result
FILE(WRITE ${CMAKE_CURRENT_BINARY_DIR}/test_helper.sh
"rm -f $3\n"
"$1 < $2\n"
"grep -q FATAL $3\n"
"if [ $? -eq 0 ]; then\n"
"echo Error\n"
"exit 1\n"
"else\n"
"exit 0\n"
"fi\n"
)

#set(float_types s d c z)
if (BUILD_SINGLE)
	list (APPEND float_types s)
endif()
if (BUILD_DOUBLE)
	list (APPEND float_types d)
endif()
if (BUILD_COMPLEX)
	list (APPEND float_types c)
endif()
if (BUILD_COMPLEX16)
	list (APPEND float_types z)
endif()
foreach(float_type ${float_types})
string(TOUPPER ${float_type} float_type_upper)
add_test(NAME "${float_type}blas1" 
  COMMAND "${CMAKE_CURRENT_BINARY_DIR}/${float_type}blat1")
add_test(NAME "${float_type}blas2" 
  COMMAND sh "${CMAKE_CURRENT_BINARY_DIR}/test_helper.sh" "${CMAKE_CURRENT_BINARY_DIR}/${float_type}blat2" "${PROJECT_SOURCE_DIR}/test/${float_type}blat2.dat" ${float_type_upper}BLAT2.SUMM)
add_test(NAME "${float_type}blas3" 
  COMMAND sh "${CMAKE_CURRENT_BINARY_DIR}/test_helper.sh" "${CMAKE_CURRENT_BINARY_DIR}/${float_type}blat3" "${PROJECT_SOURCE_DIR}/test/${float_type}blat3.dat" ${float_type_upper}BLAT3.SUMM)
endforeach()
