---
title: "Add tools"
linkTitle: "Add tools"
weight: 1
description: >
  Add a tool to neurodesktop
---

The goal of *neurodesk* is to provide users with a large choice of tools to use in their pipelines.
Use the guide below to add a tool to *neurodesktop* or *neurocontainers*. 

## Guiding principles 
To decide if a tool should be packaged in a singularity container in *neurocontainers* or be installed in the *neurodesktop* container we are currently following these guiding principles:
1) *neurodesk* is not a package manager. This means we are not distributing tools in containers that can easily be installed via a standard package manager 
2) *neurodesk* allows users to have multiple versions of tools in parallel via [lmod]( https://lmod.readthedocs.io/en/latest/), this means that if different versions of a tool can't be installed in parallel we package the tool inside a container.
3) *neurodesk* aims to provide tooling to link tools from different containers (such as workflow managers like nipype or nextflow). This means that if a tool is required to coordinate various container-tools, it should be in the *neurodesktop* container.


Examples:
|            | easy install | coordinates containers | small in size | latest version is ok | useful more most users | Conclusion                     |
|------------|--------------|------------------------|---------------|----------------------|------------------------|--------------------------------|
| git        | yes          | yes                    | yes           | yes                  | yes                    | neurodesktop                   |
| lmod       | no           | yes                    | yes           | yes                  | yes                    | neurodesktop                   |
| nipype     | yes          | yes                    | yes           | yes                  | yes                    | neurodesktop                   |
| vscode     | yes          | yes                    | yes           | yes                  | yes                    | neurodesktop                   |
| itksnap    | yes          | no                     | yes           | yes                  | yes                    | container?                     |
| convert3D  | yes          | no                     | yes           | no                   | no                     | container                      |
| fsl        | no           | no                     | no            | no                   | no                     | container                      |
| mrtrix     | no           | no                     | no            | no                   | no                     | container                      |
| freesurfer | no           | no                     | no            | no                   | no                     | container                      |


## Adding new recipes
Refer to [neurodocker](https://github.com/NeuroDesk/neurodocker) for more information on neurodocker recipes  



## Build container
To build a container:
1) Pull-in latest changes from Neurodocker upstream into our fork: https://github.com/NeuroDesk/neurodocker - add recipe to neurodocker if relevant for their project (https://github.com/NeuroDesk/neurodocker) and create a pull request to neurodocker (add new tool in a branch!)
2) Clone the neurocontainers repository https://github.com/NeuroDesk/neurocontainers
```
git clone https://github.com/NeuroDesk/neurocontainers/
```
3) Copy the directory template and rename to _newapp_ in `neurocontainers/recipes`
```
cd neurocontainers/recipes
cp -R template newapp
```
4) Modify `build.sh` in `neurocontainers/recipes/newapp` to build your application and update README.md (make sure the version is correct in the README!)
```
cd newapp
(edit build.sh as required)
(edit README.md as required)
```
5) Run update-builders.sh - This will auto-create the CI workflow for the application (or manually duplicate the template file and rename all occurances of template to _newapp_)
```
cd ../..
sh update-builders.sh
```
6) Build the container locally (e.g. running the build script with the --debug flag: https://github.com/NeuroDesk/neurocontainers/blob/master/recipes/lcmodel/build.sh)
7) updated changes in local git repository
```
git add recipes/newapp/build.sh recipes/newapp/README.md .github/workflows/newapp.yml
git config user.email "the email that you use for github"
git config user.name "your name"
git commit
```
8) Generate git personal access token (if you don’t have one already)
```
Browse to https://github.com/NeuroDesk/neurocontainers/
Press on your picture in upper right corner --> Setting --> Developer Settings --> Personal Access Token
Press on “generate personal access token”
Write something in “Notes” (doesn’t matter what, it’s for your own use)
Check “repo”
Check “Workflow”
Press “Generate Token” at the bottom
Copy the token displayed on the screen into a file, so you’ll have it later
```
9) Test the container locally, and if successful push repo to trigger the automatic build on GitHub
```
git pull
git push
```
10) Go to neurocontainers/actions. Check that the most recent workflow run in the list terminated successfully (green). Otherwise, click on it, click on “build docker”, and the line that caused the error will be highlighted
11) Check that the package shows up in repository
Go to https://github.com/orgs/NeuroDesk/packages?repo_name=neurocontainers and check that the new package is listed
12) Try to manually download the package to your Machine or VNM
```
bash /neurodesk/local/fetch_and_run.sh newapp newappversion builddate
ml newapp/newappversion
```
13) Test the new container. Run some commands, to see all is good
14) send a pull request to add the container to the apps.json file: https://github.com/NeuroDesk/neurocommand/blob/main/neurodesk/apps.json
15) (once pull request is merged this will trigger an action to build the singularity container, distribute it in all object storage locations and on CVMFS, and it will update the menus in the desktop image on the next daily build)
