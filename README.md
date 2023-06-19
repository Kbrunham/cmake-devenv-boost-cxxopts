# cmake-devenv
Template setup for CMake based C++ environment using Super Build

## Motivation
Getting the basics setup for a C++ project is a boiler plate activity but is not one that I found a satisfying answer for in existing work. I regularly develop under Linux or Windows and as a result wanted a template that worked for both. This template is my starting point for all C++ projects.

I used to have a large repository of libraries. Get's out of hand.

Using an environment variable as the helper to specify a lib get's lost and forgotten.

Why not use containers?

Types of libraries; system, in tree, environment (out of tree)


## Overview

## How to Use
```
	git clone
	git submodule update --init --recursive
	mkdir build
	cd build
	cmake .. -DPERFORM_FULL_BUILD=ON
	cmake --build . -j `nproc`
```

or

```
	git clone
	git submodule update --init --recursive
	mkdir build
	cd build
	cmake ..
	cmake --build . -j `nproc`
```

then open src/CMakeLists.txt in Visual Studio, CLion, etc. This will use the pre-build super build libraries

### Using as a template for a new project
The most common expected use of this repo is to use it when starting a new project. In this use case you clone the repo, delete the template history, and set it as the starting point for your new work.

	mkdir *mydir*
	cd *mydir*
	git clone https://github.com/Kbrunham/cmake-devenv.git .
	git remote remove origin
	git branch -M orig
	git branch main $(echo "Initial version from template" | git commit-tree HEAD^{tree})
	git checkout main
	git branch -D orig
	git remote add origin *new_remote_url*
	git push -u origin main
	git submodule update --init --recursive

#### Current src/ project submodules
   * extern/googletest v1.13.0
   * extern/cxxopts v3.1.1
## Contributing
Contributing is always welcome. Submit a pull request with any feedback.

