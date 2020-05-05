# Create AWS CLI lambda layer.
Python lambdas use this layer as an addendum to execute AWS CLI commands.

## Instructions:
###### Verify needed toos:
$ python --version
Python 3.7.0

$ pip --version
pip 18.0 from /Library/Frameworks/Python.framework/Versions/3.7/lib/python3.7/site-packages/pip (python 3.7)

$ virtualenv --version
16.2.0

### Step 1: Create work folder and set environment variables.
  a. Create folder awscli-lambda-layer in working directory.
  b. Open Terminal and cd into awscli-lambda-layer:
    i. # Automatically detects python version (only works for python3.x)
      export PYTHON_VERSION=`python3 -c 'import sys; version=sys.version_info[:3]; print("{0}.{1}".format(*version))'`
    ii. # Temporary directory for the virtual environment
      export VIRTUAL_ENV_DIR="awscli-virtualenv"
    iii. # Temporary directory for AWS CLI and its dependencies
      export LAMBDA_LAYER_DIR="awscli-lambda-layer"
    iv. # The zip file that will contain the layer
      export ZIP_FILE_NAME="awscli-lambda-layer.zip"

### Step 2: Create a virtual environment using virtualenv and activate it.
  a. # Creates a directory for virtual environment
    mkdir ${VIRTUAL_ENV_DIR}
  b. # Initializes a virtual environment in the virtual environment directory
    virtualenv ${VIRTUAL_ENV_DIR}
  c. # Changes current dir to the virtual env directory
    cd ${VIRTUAL_ENV_DIR}/bin/
  d. # Activate virtual environment
    source activate
  e. Note: Terminal prompt will now indicate that you're in a virtual envioronment. The prompt should be prepended with (awscli-virtualenv) like this:
    (awscli-virtualenv) mymac:bin username$

### Step 3: Install AWS CLI and its dependencies in the virtual environment
  pip install awscli

### Step 4: Change the path to python
  The AWS CLI executable is the file named aws and its first line provides the path to the Python interpreter. Since we're using a virtual environment, it will have a path local to the environment we created: #!/PATH_TO_WORKING_DIR/awscli-virtualenv/bin/python3.7.

  We need to replace this path with the path to the Python interpreter in the Lambda execution environment, which is #!/var/lang/bin/python.

  To replace the path to Python in the aws executable, you can run this command on MacOS.

    sed -i '' "1s/.*/\#\!\/var\/lang\/bin\/python/" aws
  
  … or this command on Linux, which has slightly different syntax because the implementation of sed on MacOS differs from that on Linux.

    sed -i "1s/.*/\#\!\/var\/lang\/bin\/python/" aws

  Alternatively, you can open aws in a text editor and replace the first line by hand.

### Step 5: Deactive virtualenv
  Run this command to deactivate the virtual environment.
    deactivate
  Note: The terminal prompt should have switched back to its normal state (no more (awscli-virtualenv) in the prompt).

### Step 6: Package AWS CLI and its depenedencies in a zip file
  The following set of commands will prepare a temporary directory for packaging AWS CLI, copy all dependencies, and zip them.

  a. # Changes current directory back to where it started
    cd ../..

  b. # Creates a temporary directory to store AWS CLI and its dependencies
    mkdir ${LAMBDA_LAYER_DIR}

  c. # Changes the current directory into the temporary directory
    cd ${LAMBDA_LAYER_DIR}

  d. # Copies aws and its dependencies to the temp directory
    cp ../${VIRTUAL_ENV_DIR}/bin/aws .
    cp -r ../${VIRTUAL_ENV_DIR}/lib/python${PYTHON_VERSION}/site-packages/ .

  e. # Zips the contents of the temporary directory
    zip -r ../${ZIP_FILE_NAME} *

### Step 8: Create layer
  Go to the Lambda console, click on Layers in the left navigation and follow the wizard to create a layer. Upload awscli-lambda-layer.zip.

### Step 9: Test 
  Create a Lambda function using the same version of Python that was used for packaging AWS CLI.
  
  After the function is created, in Designer, click on Layers, click Add layer and select the layer you've just created.


# Credit: 
  https://bezdelev.com/hacking/aws-cli-inside-lambda-layer-aws-s3-sync/
