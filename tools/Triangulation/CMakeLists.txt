# Include this directory
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})

# Enumerate source files
FILE(GLOB SRC_FILES *.cpp *.inl)
FILE(GLOB INC_FILES *.h)

SET(DEPENDENT_LIBS "")

# IGL and CGAL
IF (LIBIGL_FOUND AND CGAL_FOUND)
    INCLUDE_DIRECTORIES(${LIBIGL_INCLUDE_DIRS})
    INCLUDE(${CGAL_USE_FILE})
    ADD_DEFINITIONS(-DWITH_IGL)
    ADD_DEFINITIONS(-DWITH_CGAL)
    LIST(APPEND DEPENDENT_LIBS ${CGAL_LIBRARIES})
    ADD_SUBDIRECTORY(IGL)
    ADD_SUBDIRECTORY(CGAL)
ENDIF (LIBIGL_FOUND AND CGAL_FOUND)

# Shewchuk's Triangle
IF (TRIANGLE_FOUND)
    INCLUDE_DIRECTORIES(${Triangle_INCLUDE_DIRS})
    LIST(APPEND DEPENDENT_LIBS ${Triangle_LIBRARIES})
    ADD_DEFINITIONS(-DWITH_TRIANGLE)
    ADD_SUBDIRECTORY(ShewchukTriangle)
ENDIF (TRIANGLE_FOUND)

# GeoGram
IF (GEOGRAM_FOUND)
    INCLUDE_DIRECTORIES(${GEOGRAM_INCLUDE_DIRS})
    LIST(APPEND DEPENDENT_LIBS ${GEOGRAM_LIBRARIES} lib_geogram_base)
    ADD_DEFINITIONS(-DWITH_GEOGRAM)
    ADD_SUBDIRECTORY(GeoGram)
ENDIF (GEOGRAM_FOUND)

ADD_LIBRARY(lib_triangulation SHARED ${SRC_FILES} ${INC_FILES})
SET_TARGET_PROPERTIES(lib_triangulation PROPERTIES OUTPUT_NAME "triangulation")
TARGET_LINK_LIBRARIES(lib_triangulation Mesh ${DEPENDENT_LIBS})

IF(NOT WITHOUT_SWIG)
    set(SWG_FILES PyTriangulation.i)

    # Generate wrapper for python
    SET_SOURCE_FILES_PROPERTIES(${SWG_FILES} PROPERTIES CPLUSPLUS ON)
    SET_SOURCE_FILES_PROPERTIES(${SWG_FILES} PROPERTIES CMAKE_SWIG_FLAGS "-includeall")
    SWIG_ADD_MODULE(PyTriangulation python ${SWG_FILES} ${INC_FILES})
    SWIG_LINK_LIBRARIES(PyTriangulation ${PYTHON_LIBRARIES} Mesh lib_MeshUtils
        lib_triangulation ${DEPENDENT_LIBS})

    # Boudle lib, swig wrapper into a single target.
    # Usage: make triangulation
    ADD_CUSTOM_TARGET(triangulation DEPENDS lib_triangulation
        _PyTriangulation)
ELSE(NOT WITHOUT_SWIG)
    ADD_CUSTOM_TARGET(triangulation DEPENDS lib_triangulation)
ENDIF(NOT WITHOUT_SWIG)