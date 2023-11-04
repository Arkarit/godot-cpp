#!/usr/bin/env python

import os
import platform
import sys
import subprocess
from binding_generator import scons_generate_bindings, scons_emit_files
from importlib.util import spec_from_file_location, module_from_spec


EnsureSConsVersion(4, 0)


try:
    Import("env")
except:
    # Default tools with no platform defaults to gnu toolchain.
    # We apply platform specific toolchains via our custom tools.
    env = Environment(tools=["default"], PLATFORM="")

def _helper_module(name, path):
    spec = spec_from_file_location(name, path)
    module = module_from_spec(spec)
    spec.loader.exec_module(module)
    sys.modules[name] = module
    # Ensure the module's parents are in loaded to avoid loading the wrong parent
    # when doing "import foo.bar" while only "foo.bar" as declared as helper module
    child_module = module
    parent_name = name
    while True:
        try:
            parent_name, child_name = parent_name.rsplit(".", 1)
        except ValueError:
            break
        try:
            parent_module = sys.modules[parent_name]
        except KeyError:
            parent_module = ModuleType(parent_name)
            sys.modules[parent_name] = parent_module
        setattr(parent_module, child_name, child_module)

_helper_module("methods", "methods.py")
import methods

env.PrependENVPath("PATH", os.getenv("PATH"))

# Custom options and profile flags.
customs = ["custom.py"]
try:
    customs += Import("customs")
except:
    pass
profile = ARGUMENTS.get("profile", "")
if profile:
    if os.path.isfile(profile):
        customs.append(profile)
    elif os.path.isfile(profile + ".py"):
        customs.append(profile + ".py")
opts = Variables(customs, ARGUMENTS)
cpp_tool = Tool("godotcpp", toolpath=["tools"])
cpp_tool.options(opts, env)

opts.Add(BoolVariable("vsproj", "Generate a Visual Studio solution", False))
opts.Add("vsproj_name", "Name of the Visual Studio solution", "godot_cpp")
opts.Update(env)


Export("env")
if env["vsproj"]:
    env.vs_incs = []
    env.vs_srcs = []

Help(opts.GenerateHelpText(env))

# Detect and print a warning listing unknown SCons variables to ease troubleshooting.
unknown = opts.UnknownVariables()
if unknown:
    print("WARNING: Unknown SCons variables were passed and will be ignored:")
    for item in unknown.items():
        print("    " + item[0] + "=" + item[1])

scons_cache_path = os.environ.get("SCONS_CACHE")
if scons_cache_path is not None:
    CacheDir(scons_cache_path)
    Decider("MD5")


if env["vsproj"]:
    if os.name != "nt":
        print("Error: The `vsproj` option is only usable on Windows with Visual Studio.")
        Exit(255)
    #env["CPPPATH"] = [Dir(path) for path in env["CPPPATH"]]
    methods.generate_vs_project(env, ARGUMENTS, env["vsproj_name"])
    methods.generate_cpp_hint_file("cpp.hint")
	



cpp_tool.generate(env)
library = env.GodotCPP()

Return("env")
