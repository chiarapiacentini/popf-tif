PROJECT(popf2)

set( CMAKE_MODULE_PATH    ${CMAKE_MODULE_PATH}
                          ${CMAKE_CURRENT_SOURCE_DIR}
                          ${CMAKE_CURRENT_SOURCE_DIR}/cmake/modules
                          ${CMAKE_CURRENT_SOURCE_DIR}/../cmake/modules)


set(DOXYFILE_TARGET_SUFFIX popf)
include(UseDoxygen OPTIONAL)


find_package(Clp)
find_package(CoinUtils)
find_package(Cbc)
find_package(Cgl)
find_package(OSI)


set(CMAKE_CXX_FLAGS "-Wall ${CMAKE_CXX_FLAGS}")

if(STATIC_POPF)
set(CMAKE_CXX_LINK_FLAGS "-static ${CMAKE_CXX_LINK_FLAGS}")
endif(STATIC_POPF)

include_directories(${val_SOURCE_DIR} ${val_SOURCE_DIR}/parsing ${val_BINARY_DIR} ${CLP_INCLUDES}/coin ${CBC_INCLUDES}/coin ${CPLEX_INCLUDES} ${CONCERT_INCLUDES})

set(popf2_build_srcs 
    ExternalSolver.cpp 
    LookAhead.cpp 
    ModifiedSearch.cpp
    Color.cpp 
    TimewindowsChecker.cpp 
    DotSearchSpace.cpp
    compressionsafescheduler.cpp
    totalordertransformer.cpp
    partialordertransformer.cpp
    FFSolver.cpp
    FFEvent.cpp
    globals.cpp
    lpscheduler.cpp
    RPGBuilder.cpp
    RPGBuilderAnalysis.cpp
    RPGBuilderEvaluation.cpp
    RPGBuilderNumerics.cpp
    temporalanalysis.cpp
    minimalstate.cpp
    temporalconstraints.cpp
    numericanalysis.cpp
    solver.cpp)



add_library(POPF2Common STATIC ${popf2_build_srcs})
target_link_libraries(POPF2Common ParsePDDL Inst)

add_library(POPF3Common STATIC ${popf2_build_srcs})
target_link_libraries(POPF3Common ParsePDDL Inst)
SET_TARGET_PROPERTIES(POPF3Common PROPERTIES COMPILE_FLAGS "-DPOPF3ANALYSIS")


if (CLP_INCLUDES AND CBC_INCLUDES)

    ADD_EXECUTABLE(popf3-clp popfMain.cpp solver-clp.cpp)
    
    target_link_libraries(popf3-clp
        POPF3Common
        ${CBC_SOLVER_LIBRARY} ${CBC_LIBRARY} ${CGL_LIBRARIES} ${OSICLP_LIBRARIES} ${OSI_LIBRARIES} ${CLP_LIBRARIES}  ${COINUTILS_LIBRARIES} bz2 z dl
    )
    SET_TARGET_PROPERTIES(popf3-clp PROPERTIES COMPILE_FLAGS "-DPOPF3ANALYSIS")

endif(CLP_INCLUDES AND CBC_INCLUDES)

if(CPLEX AND ILOCPLEX AND CONCERT AND CPLEX_INCLUDES AND CONCERT_INCLUDES)

    ADD_EXECUTABLE(popf3-cplex popfMain.cpp solver-cpx.cpp)
    
    target_link_libraries(popf3-cplex    
            POPF3Common
            ${ILOCPLEX} ${CPLEX} ${CONCERT} pthread
    )
    SET_TARGET_PROPERTIES(popf3-cplex PROPERTIES COMPILE_FLAGS "-DPOPF3ANALYSIS")
    
endif(CPLEX AND ILOCPLEX AND CONCERT AND CPLEX_INCLUDES AND CONCERT_INCLUDES)

enable_testing()
add_subdirectory(testcases)

