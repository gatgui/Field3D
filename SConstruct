import os
import glob
import excons
from excons.tools import hdf5
from excons.tools import ilmbase
from excons.tools import boost
from excons.tools import dl

# FIELD3D_STATIC

defs = []

static = (excons.GetArgument("static", 0, int) != 0)
if static:
   defs.append("FIELD3D_STATIC")

math_inc = excons.GetArgument("math-inc", None)
if math_inc:
   defs.extend(["FIELD3D_CUSTOM_MATH_LIB", 'FIELD3D_MATH_LIB_INCLUDE=\\"' + math_inc + '\\"'])
else:
   math_inc = "StdMathLib.h"

extra_ns = excons.GetArgument("extra-ns", None)
if extra_ns:
   defs.append("FIELD3D_EXTRA_NAMESPACE=%s" % extra_ns)

verbose = (excons.GetArgument("verbose", 0, int) != 0)

headers = glob.glob("export/*.h")
headers.remove("export/Types.h")

targets = [
   {"name": "Field3D",
    "type": "staticlib" if static else "sharedlib",
    "defs": defs,
    "incdirs": ["export"],
    "srcs": glob.glob("src/*.cpp"),
    "install": {"include/Field3D": headers},
    "custom": [hdf5.Require(hl=False, verbose=verbose),
               ilmbase.Require(ilmthread=False, iexmath=False),
               boost.Require(libs=["system", "thread"]),
               dl.Require]},
   {"name": "f3dinfo",
    "type": "program",
    "defs": defs,
    "incdirs": ["export"],
    "libs": ["Field3D"],
    "srcs": ["apps/f3dinfo/main.cpp"],
    "custom": [hdf5.Require(hl=False),
               ilmbase.Require(ilmthread=False, iexmath=False),
               boost.Require(libs=["program_options", "system", "regex"]),
               dl.Require]},
   {"name": "f3dmakemip",
    "type": "program",
    "defs": defs,
    "incdirs": ["export"],
    "libs": ["Field3D"],
    "srcs": ["apps/f3dmakemip/main.cpp"],
    "custom": [hdf5.Require(hl=False),
               ilmbase.Require(ilmthread=False, iexmath=False),
               boost.Require(libs=["program_options", "system", "thread"]),
               dl.Require]},
   {"name": "f3dsample",
    "type": "program",
    "defs": defs,
    "incdirs": ["export"],
    "libs": ["Field3D"],
    "srcs": ["apps/f3dsample/main.cpp"],
    "custom": [hdf5.Require(hl=False),
               ilmbase.Require(ilmthread=False, iexmath=False),
               boost.Require(libs=["program_options", "system"]),
               dl.Require]}
]

env = excons.MakeBaseEnv()

excons.DeclareTargets(env, targets)

def bakeMathLibHeader(target, source, env):
   if len(target) != 1 or len(source) != 1:
      print "Wrong number of arguments to bakeTypesIncludeFile"
      return
   
   out = open(str(target[0]), "w")
   inFile = open(str(source[0]))
   skip = False
   
   for line in inFile.readlines():
      if not skip and "#ifdef FIELD3D_CUSTOM_MATH_LIB" in line:
         skip = True
         newLine = '#include "' + math_inc + '"\n'
         out.writelines(newLine)
      
      if not skip:
         out.writelines(line)
      
      if skip and "#endif" in line:
         skip = False

env.Append(BUILDERS={"BakeMathLibHeader": Builder(action=bakeMathLibHeader, suffix=".h", src_suffix=".h")})

if excons.no_arch:
   headerDir = "%s/%s/include/Field3D" % (excons.out_dir, excons.mode_dir)
else:
   headerDir = "%s/%s/%s/include/Field3D" % (excons.out_dir, excons.mode_dir, excons.arch_dir)

bakeTarget = env.BakeMathLibHeader(os.path.join(headerDir, "Types.h"), "export/Types.h")
env.Depends("Field3D", bakeTarget)

Default(["Field3D", "f3dinfo", "f3dsample", "f3dmakemip"])
