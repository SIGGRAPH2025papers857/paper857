# SIGGRAPH 2025 papers_857
Anonymous repository for paper submission: SIGGRAPH 2025, paper 857



We will update detailed instructions, more organized code, and a Docker image upon final submission.



## System Requirements

- **Operating System:** Windows 11 or Ubuntu 20.04
- **Python:** 3.8
- Dependencies:
  - `numpy`
  - `open3d==0.18.0`
  - `scikit-learn`
  - `scipy`
  - `trimesh==4.1.0`

Clone this project:

```
https://github.com/SIGGRAPH2025papers857/papers857.git
```



## Step 1: Static and Dynamic Parts Identification

Navigate to TSMC root:

```
cd ../TSMC
```

Run the identification script `identify_regions.ipynb`

## Step 2: Volume Tracking

Volume tracking is written in C# and .NET 7.0

Navigate to the root directory:

```
cd ./volume-tracking
```

Build the project:

```
dotnet build -c release
```

Run the tracking process:

```
dotnet ./bin/Client.dll ./config/max/<config_file.xml>
```

Volume tracking results are saved in the `<outDir>` folder:

- `.xyz` files: coordinates of volume centers
- `.txt` files: transformations of centers between frames

## Step 3: Generate Reference Centers Using Multi-Dimensional Scaling (MDS)

Navigate to TSMC root:

```
cd ../TSMC
```

Run the MDS script:

```
python ./get_reference_center.py
```

## Step 4: Compute Transformation Dual Quaternions

Then, we compute the transformations for each center, mapping their original positions to the reference space, along with their inverses. These transformations are then used to deform the mesh surface based on the movement of volume centers.

```
python ./get_transformation.py
```

## Step 5: Create Volume-Tracked, Self-Contact-Free Reference Mesh

For Linux, switch to .NET 5.0.

```
sudo apt-get install -y dotnet-sdk-5.0
sudo apt-get install -y aspnetcore-runtime-5.0
```

Navigate to the `center-affinity-deformation` directory and build:

```
cd ../center-affinity-deformation
dotnet new globaljson --sdk-version 5.0.408 
dotnet build TVMEditor.sln --configuration Release --no-incremental
```

(For Windows the writer used .NET 8.0 and there is no need to install .NET 5.0 and run `dotnet new globaljson --sdk-version 5.0.408 `. If you encounter error regarding .NET version, try to install the correct version on your machine.)

Run the center affinity deformation:

for Windows:

```
TVMEditor.Test/bin/Release/net5.0/TVMEditor.Test.exe
```

for Linux:

```
TVMEditor.Test/bin/Release/net5.0/TVMEditor.Test
```

Navigate to TSMC root again:

```
cd ../TSMC
```

Extract the reference mesh:

```
python ./extract_reference_mesh.py
```

## Step 6: Deform Reference Mesh to Each Mesh in the Group

Navigate to the `center-affinity-deformation` directory,

```
cd ../center-affinity-deformation
```

Then run:

```
TVMEditor.Test/bin/Release/net5.0/TVMEditor.Test.exe
```

For Linux:

```
TVMEditor.Test/bin/Release/net5.0/TVMEditor.Test
```

## Step 7: Compute Displacement Fields

Navigate to ./TSMC again:

```
cd ../TSMC
```

```
python ./get_displacements.py
```

The displacement fields are stored as `.ply` files. For compression, Draco is used to encode both the reference mesh and displacements.

## Step 8: Compression and Evaluation

Navigate to TSMC root:

```
cd ..
```

Clone and build Draco:

```
git clone https://github.com/google/draco.git
cd ./draco
mkdir build
cd build
```

On Windows:

```
cmake ../ -G "Visual Studio 17 2022" -A x64
cmake --build . --config Release
```

On Linux:

```
cmake ../
make
```

Draco paths (please change based on your project):

- Encoder: `./draco/build/Release/draco_encoder.exe` / `./draco/build/draco_encoder` 
- Decoder: `./draco/build/Release/draco_decoder.exe` / `./draco/build/draco_decoder` 

Navigate to TSMC:

```
cd ../../TSMC
```

Run the evaluation:

```
python ./evaluation.py
```


