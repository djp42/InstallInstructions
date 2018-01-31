# Installing Blake's ngsim_env and gail
- These are the install instructions along with logfiles for [ngsim_env](https://github.com/wulfebw/ngsim_env) followed by gail 
- ngsim section adapted from Blake's ngsim install [instructions](https://github.com/wulfebw/ngsim_env/blob/master/docs/install.md) then added to by Raunak on Jan 26
- Hgail adapted from Derek's install instructions and then added to by Raunak on Jan 25

# Installation instructions for ngsim_env 
```bash
# install miniconda
wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh
# answer yes to everything
sh ./Miniconda3-latest-Linux-x86_64.sh
rm Miniconda3-latest-Linux-x86_64.sh
source ~/.bashrc

# install rllab
git clone https://github.com/rll/rllab.git
cd rllab
# this takes a while
conda env create -f environment.yml
conda env update
# activate the conda environment
source activate rllab3
python setup.py develop
cd ..
```

## Install julia
```bash
# install our own version of julia
wget https://julialang-s3.julialang.org/bin/linux/x64/0.6/julia-0.6.2-linux-x86_64.tar.gz
tar -xf julia-0.6.2-linux-x86_64.tar.gz
rm julia-0.6.2-linux-x86_64.tar.gz

# add this line to avoid a pyjulia issue where it uses the wrong julia
# this makes it so that by default we use this julia install
echo "export PATH=$(pwd)/julia-d386e40c17/bin:\$PATH" >> ~/.bashrc
source ~/.bashrc
```

## ngsim_env

### julia
```bash
source activate rllab3 # this is probably not necessary, but just in case
git clone https://github.com/wulfebw/ngsim_env.git
# this takes a long time
julia ngsim_env/julia/deps/build.jl

# this avoids a bug
echo "using Iterators" >> ~/.juliarc.jl
# avoiding a PyPlot bug
echo "export LD_PRELOAD=${HOME}/.julia/v0.6/Conda/deps/usr/lib/libz.so" >> ~/.bashrc 
source ~/.bashrc
# manually add AutoEnvs
echo "push!(LOAD_PATH, \"$(pwd)/ngsim_env/julia/AutoEnvs\")" >> ~/.juliarc.jl
source activate rllab3
# enter a julia interpreter
julia
  # set python path (replace with your miniconda3 install location)
  >>ENV["PYTHON"] = "/home/wulfebw2/miniconda3/envs/rllab3/bin/python"
  # if any of this raises a bug, fix it before moving on
  # this installs the julia-internal conda referenced above
  >>using PyCall
  >>using PyPlot
    # takes a while
      # If this errors ImportError('No module named mpl_toolkits.mplot3d',), you need to upgrade matplotlib
  >>quit()
pip install --upgrade matplotlib
# Check the install went through correctly
python
    >>>import mpl_toolkits
    >>>quit()

# Open up Julia interpreter again and try using PyPlot again
julia
  >> using PyPlot
  >>using AutoEnvs
  >>quit()
```
Here is a [logfile](logFiles/installLog_ngsim) for the above. Next, we will get the NGSIM data and run a few tests with julia and python to make sure everything is fine

### download NGSIM data
```bash
##Get the data
cd ~/.julia/v0.6/NGSIM/data
wget https://github.com/sisl/NGSIM.jl/releases/download/v1.0.0/data.zip
unzip data.zip

##Fix a trajectory converstion script
cd ../src
#change line 205 of .julia/v0.6/NGSIM/src/trajdata.jl" to outpath = Pkg.dir("NGSIM", "data", "trajdata_"*splitdir(filename)[2])

##Create trajectories from the data
cd ../data
julia
  >> using NGSIM
  >> convert_raw_ngsim_to_trajdatas()
  >> quit()
```

### run julia tests

```bash
# run the julia tests
# if you don't get an error, everything works with julia
# it will take a few minutes because it's creating some cached files
cd ngsim_env/julia/test
julia runtests.jl
cd ../../..
```

### run python tests
```bash
# install the python components of the package
source activate rllab3 # this is probably not necessary, but just in case
cd ngsim_env/python
python setup.py develop
pip install julia
cd tests
# one of the these tests will fail if you don't have hgail installed

python runtests.py
  # if you get a segfault, need to delete the PyCall.jl cache file

  cd ~/.julia/lib/v0.6
  rm PyCall.jl
  # Check
  python
    >>>import julia
    >>>quit()

# After removing PyCall.jl let's try the test again (all this is assuming you got a seg fault)
cd ~/ngsim_env/python/tests
python runtests.py
  # If you get the error: 
  # ERROR: test_vectorized_ngsim_env (unittest.loader._FailedTest)
  # Intel MKL FATAL ERROR: Cannot load libmkl_avx2.so or libmkl_def.so

  conda install nomkl numpy scipy scikit-learn numexpr
  # Found the fix https://stackoverflow.com/questions/36659453/intel-mkl-fatal-error-cannot-load-libmkl-avx2-so-or-libmkl-def-so
  # Answer by libphy
  # Then run the test again and it should be fine

```
Here is a [logfile](logFiles/ngsim_test_log) for the above

# Installation instructions for hgail
```bash
cd ~
git clone https://github.com/wulfebw/hgail
source activate rllab3
cd hgail
python setup.py develop
cd tests python runtests.py
cd ~/ngsim_env
mkdir data
mkdir data/trajectories
mkdir data/experiments
cd scripts
julia
  >> Pkg.checkout("AutomotiveDrivingModels", "lidar_sensor_optimization")
  >> quit()

cd ~/ngsim_env/scripts

julia extract_ngsim_demonstrations.jl
cd imitation
python imitate.py [options here]
```
See installation ![logFile](logFiles/installLog_hgail)
