# cran-publish-action
Simple github action to publish R packages in a CRAN-like repository. This action assumes that all the R packages (source or binary) have been uploaded through upload-artifact action.

I made this following this guide :
https://docs.github.com/en/actions/creating-actions/creating-a-composite-action

**Important**: Only support **Linux** docker container. But if you want to publish **Windows** or **Mac** R binary packages, the idea is to build under **Windows**/**Mac** and publish under **Linux** usng this action (see usage).

## Requirements
- A local or a web server for your CRAN repository
- The server must accept SSH connections
- An R version with (drat)[https://cran.r-project.org/web/packages/drat/index.html] package must be installed on the server

## Input variables
* ```host``` - CRAN server where you can execute SSH commands
* ```username``` - Username for SSH connection
* ```password:``` - Password for SSH connection
* ```repo-path``` - Full path to the CRAN repository in the server

## Usage
Build Windows R packages and publish them to a CRAN

   name: build and publish R packages (windows)
   on:
     workflow_dispatch:
       inputs:
         logLevel:
           description: 'Manual'
   
   jobs:
     
     build:
       runs-on: windows-2019
       strategy :
         matrix:
           r_version: ["4.2.3","4.3.0"]
           
       steps:
       - uses: actions/checkout@v3
   
       - name: Setup R Version
         uses: r-lib/actions/setup-r@v2
         with:
           r-version: ${{matrix.r_version}}.
           
       - name : Create R Package and store filename in github environment
         run : |
           R CMD INSTALL --build --no-multiarch .
           $PKG_PATH = Get-ChildItem -Path "*.zip" -File
           echo "MY_PKG=$PKG_PATH" | Out-File -FilePath $Env:GITHUB_ENV -Encoding utf8 -Append
   
       - uses: actions/upload-artifact@v3
         # Use specific artifact identifier for publishing all R versions
         with:
           name: windows-package-${{matrix.r_version}}
           path: ${{ env.MY_PKG }}
           
       
     publish:
       needs: build
       # Only ubuntu can upload via ssh
       runs-on: ubuntu-latest
       
       steps:
       - uses: fabien-ors/cran-publish-action@v9
         with:
           host: ${{ secrets.CRAN_HOST }}
           username: ${{ secrets.CRAN_USR }}
           password: ${{ secrets.CRAN_PWD }}
           repo-path: "/path/to/cran/on/the/server"

        
