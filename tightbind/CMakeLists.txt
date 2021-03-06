cmake_minimum_required(VERSION 2.6)

project(yaehmop C)

# Source files
set(YAEHMOP_SRCS
  abfns.c
  avg_props.c
  bands.c
  charge_mat.c
  chg_it.c
  COOP_stuff.c
  distance_mat.c
  DOS_stuff.c
  electrostat.c
  fileio.c
  FMO_stuff.c
  genutil.c
  geom_frags.c
  globals.c
  K_hamil.c
  K_overlap_mat.c
  kpoints.c
  lovlap.c
  main.c
  matrices.c
  memory.c
  mod_mulliken.c
  mov.c
  muller.c
  mulliken.c
  netCDF_support.c
  new3_fileio.c
  postprocess.c
  princ_axes.c
  R_hamil.c
  R_overlap_mat.c
  recip_space.c
  solid_symmetry.c
  symmetry.c
  transforms.c
  walsh.c
  xtal_coords.c
  zetas.c
  Zmat.c
)

# Should we use blas and lapack?
option(USE_BLAS_LAPACK
       "Whether or not to link to BLAS and LAPACK for much faster matrix
       multiplication"
       OFF)

option(BUILD_STATIC
       "Whether or not to build the entire executable as static"
       OFF)


# If we aren't using blas and lapack, we must build these as well
if(NOT USE_BLAS_LAPACK)
  set(YAEHMOP_SRCS
    ${YAEHMOP_SRCS}
    cboris.c
    diag.c
  )
endif(NOT USE_BLAS_LAPACK)


# We link to the static library to avoid an error

# The location of the parameters file needs to be defined
set(PARM_FILE_LOC ${yaehmop_SOURCE_DIR}/eht_parms.dat)
message("-- Parameter file is currently set to ${PARM_FILE_LOC}")
message("-- Change the parameter file with -DPARM_FILE_LOC=<location>")
add_definitions(-DEHT_PARM_FILE=\"${PARM_FILE_LOC}\")

# This just adds an underscore after some function names in the source...
add_definitions(-DUNDERSCORE_FORTRAN)

# Add the executable and link to the C math library
add_executable(bind ${YAEHMOP_SRCS})

# If we are using LAPACK and BLAS, link to them
if(USE_BLAS_LAPACK)
  # Should we perform static or dynamic linkage? Default is dynamic
  option(STATIC_BLAS_LAPACK
         "If we are using blas and lapack, whether to link them statically "
         OFF)

  # If we are building a static executable, we must set this to on
  if(BUILD_STATIC)
    message("-- Linking to blas and lapack statically since BUILD_STATIC is on")
    set(STATIC_BLAS_LAPACK ON)
  endif(BUILD_STATIC)

  if(STATIC_BLAS_LAPACK)
    message("-- Using static linkage for blas and lapack")

    # Apple has to find the package regardless...
    if(APPLE)
      message("-- NOTE: For Apple, you may need to remove all libquadmath")
      message("--       files except libquadmath.a to get it to link")
      message("--       to libquadmath statically.")
      find_package(LAPACK REQUIRED)
      target_link_libraries(bind ${LAPACK_LIBRARIES})
    else(APPLE)
      message("-- Attempting to link to liblapack.a and libblas.a")
      message("-- Note that we must also link to gfortran for static linking")
      # We have to statically link to lapack and blas
      target_link_libraries(bind liblapack.a libblas.a)
    endif(APPLE)

    # Link these as well if we are not using MINGW
    if(NOT MINGW)
      target_link_libraries(bind libgfortran.a libquadmath.a)
    endif(NOT MINGW)
  else(STATIC_BLAS_LAPACK)
    # If we are just linking to the dynamic libraries, cmake can find them
    message("-- Dynamically linking to Lapack and Blas libraries.")
    message("-- To statically link, use -DSTATIC_BLAS_LAPACK=ON")
    find_package(LAPACK REQUIRED)
    message("-- Lapack found")
    message("-- Lapack and Blas libraries are: ${LAPACK_LIBRARIES}")
    target_link_libraries(bind ${LAPACK_LIBRARIES})
  endif(STATIC_BLAS_LAPACK)
  # This is needed for the code
  add_definitions(-DUSE_LAPACK)
endif(USE_BLAS_LAPACK)

if(BUILD_STATIC)
  set(BUILD_SHARED_LIBRARIES OFF)
  # We are probably going to need to change this if we allow for MSVC
  # compilation. But we can cross that bridge later - we don't need MSVC
  # compilation right now.

  # Apple should not compile with the -static flag
  if(APPLE)
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static-libgcc")
  else(APPLE)
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static")
  endif(APPLE)

endif(BUILD_STATIC)

# Link to the C Math library
target_link_libraries(bind m)
