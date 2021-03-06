# SVT-HEVC Test Script Overview

### Python

The test script is written in Python.  Supported Python versions include 2.7 and 3.7
	

### Operating Systems

Test cases can be run on both Linux and Windows platforms.



### Paths

The following relative paths are defined in the script:\
ENC_PATH   = folder where the SvtHevcEncApp executable can be found - default: "encoders"\
BIN_PATH   = folder where the created bitstreams are stored - default: "bitstreams"\
YUV_PATH   = folder where the input yuv media file are stored - default: "yuvs"\
TOOLS_PATH = folder where the tools are located (e.g. reference decoder, MCTS decoder) - default: "tools"\
These folders should be created prior to running the script.\
The bitstream folder should be empty and cleared for each run.\
Note: You should build the SVT encoder and place the executable in the folder specified under "ENC_PATH"\
Note: Download the reference decoder from https://hevc.hhi.fraunhofer.de/ and place in the folder specified under "TOOLS_PATH"\
Note: The MCTS check decoder can be found at: https://github.com/kelvinhu325/HM/tree/mcts_check .  The binary should be build and place in the folder specified under "TOOLS_PATH"



### Validation Test Modes

VALIDATION_TEST_MODE defines which of the 3 validation test modes that the script runs in:\
0 = Fast - Used to quickly verify code check-ins.  Should take around 2 hours\
1 = Nightly - Used as a daily stability check-ins_support. Should take around 18 hours\
2 = Full - Used to verify releases of the products.  Should take around 2 days.\
Note: Estimated time is based on tests ran on Intel Xeon Gold 6140 CPU w/ 94.7 GB memory.\
The script modes differ by the total number of tests that are run.



### COLOR_MODE

COLOR_MODE defines which subset of media files are used in the test

0 = P420, P422, and P444\
1 = P420 Only\
2 = P422 Only\
3 = P444 Only



### QP/VBR

QP_VBR_MODE defines how the quantization parameter, variable bitrate, and constant rate factor parameters are specified in the tests:

0 =QP, VBR, and CRF parameters are used\
1 = QP Only\
2 = VBR Only\
3 = CRF Only\
4 = QP and VBR Only
	

### Encoder Modes

SPEED_ENC_MODES is a list variable that defines which encoding modes (0-11) are used in the speed tests. It defaults to [0,6,9]

ENC_MODES is a list variable that defines which encoding modes (0-11) are used in each of the validation test modes.	

### Number of Frames

NUM_FRAMES is a variable that defines the number of frames encoded in each test. It default to 20
	

### Quantization Iterations

MIN_QP and MAX_QP define the range of quantization that the tests can be run with.\
QP_ITERATIONS defines the number of random quantization values used in that range.\
MIN_QP defaults to 30\
MAX_QP defaults to  50\
QP_ITERATIONS defaults to 1\
QP iteration is only used when the QP parameter is being used



### Variable Bitrate Iterations

MIN_BR and MAX_BR define the range of bitrates (bytes/second) that the tests can be run with.\
VBR_ITERATIONS defines the number of variable bitrate values used in that range.\
MIN_BR defaults to 1000\
MAX_BR defaults to 10000000\
VBR_ITERATIONS defaults to 1 for fast and nightly test modes, and 2 for full test modes\
VBR iteration is only used when VBR parameter is being used



### CRF Iterations

MIN_CRF and MAX_CRF define the range of rate factors that the tests can be run with.\
CRF_ITERATIONS defines the number of iterations to set when Constant Rate Factor testing is specified.



### Search Area Iterations

SA_ITER defines the size of the search area used in the tests
	

### Look Ahead Distance Iterations

LAD_ITER defines the number of look ahead iterations that are used in the tests



### Intra Period Iterations

INTRA_PERIOD_ITER defines the number of intra period iterations that are used in the tests



### MCTS Iterations

MCTS_ITER defines the number of motion constrained tileset iterations that are used in the tests



### Width Height Iterations

WH_ITER defines the number of width/height pairs are used in the tests\
MIN_WIDTH and MAX_WIDTH define the range of widths (default: 832-4096)\
MIN_HEIGHT and MAX_HEIGHT define the range of heights (default: 480-2304)



### Debugging

Setting DEBUG_MODE to a non-zero value will allow the script to run and configure each test to be run, but not actually run the test.



### Configuration

TEST_CONFIGURATION can be set to 0 or 1\
        0 = Validation Test - Verify that encoder parameters are functioning properly\
        1 = Speed Test - Check the speed of encoding

Both set of tests use different set of media files (VALIDATION_TEST_SEQUENCES
and SPEED_TEST_SEQUENCES).

Validation tests tabulate sums of the number of test that are run and the 
number of tests that pass.  The total time taken to run the full set of tests is
calculated.

Speed tests are written into a batch file (speed_script.bat/speed_script.sh)
which can executed outside of this batch file.

### Adding a validation test

Here are the basic steps to follow when adding a test to validate the correct processing of an encoder parameter.  You can refer to existing tests functions (e.g. sao_test) for reference on creating a new test function.
	

1. Add a function that runs a test at the bottom of the section labeled FUNCTIONAL TESTS.

   - The name of the function should reflect the test(s) scope (e.g. myparameter_test)

   - The two parameters to the function should be:

     - self - standard Python instance variable

     - seq_list - list of sequences (videos) that are to be run
       These parameters are not used in the body of the function, but are passed in the \
       return when the function ends.

       Example Function signature:

       ```python
       def myparameter_test(self,seq_list)
       ```

   - Add a body to function

     - create a variable that defines the name of the test (e.g. test_name = 'myparameter_test')

     - create a dictionary that defines parameters and a list of values.  More than one parameter can be included in the dictionary (e.g.)

       ```python
       combination_test_params = { 'MyParameterName'     : [0, 1],
                                   'MyOtherParameterName'     : [0, 1],
       }					   
       ```

     - make sure that all parameters used in the dictionary are also included in the default_tokens dictionary created in the get_param_tokens	function (e.g.)

       ```python
        default_tokens = {
        ...
        'MyParameterName' : '-mp',
        'MyOtherParameterName' : '-mop',
        ...
       ```

     - Add a return statement to the function which calls run_functional_tests passing  the sequence list, test name, and test parameters (e.g.)

       ```python
        return self.run_functional_tests(seq_list, test_name, combination_test_params)
       ```

   - Here's the resulting function:

     ```python
     def myparameter_test(self,seq_list){
         # Test specific parameters:
         test_name = 'myparameter_test'
         combination_test_params = { 'MyParameterName'     : [0, 1],
                                     'MyOtherParameterName'     : [0, 1],
                                   }
         # Run tests
         return self.run_functional_tests(seq_list, test_name, combination_test_params)
     ```

2. Add a call to the new function at the end of the set of validation tests that are run.  In the function run_validation_test, add these lines that call your function and tabulate the num_tests and num_passed counters (e.g.)

   ```python
    num_tests, num_passed = self.myparameter_test(seq_list)
    total_tests = total_tests + num_tests
    total_passed = total_passed + num_passed
   ```

3. Try running the script in debug mode.  There should be a txt file with a name based on your test name (e.g. myparameter_test.txt).  Open the text file and verify that  the individual tests run match with the parameters you've specified in your function.

   

### How to use the test script:

1. Make sure ENC_PATH, BIN_PATH,YUV_PATH, and TOOLS_PATH folders are created relative to the location of  where the script is located.  The bitstream folder should be empty and cleared for each run.

2. Build SVT encoder and place the executable in the folder specified under "ENC_PATH"

3. Download the reference decoder from https://hevc.hhi.fraunhofer.de/

4. Build the reference decoder and place it in the folder specified under "TOOLS_PATH"(Windows:TAppDecoder.exe , Linux: TAppDecoder)

5. Download the MCTS decoder from https://github.com/kelvinhu325/HM/tree/mcts_check

6. Build the MTCS decoder and place it in the folder specified under "TOOLS_PATH"(Windows:MCTS_TAppDecoder.exe , Linux: MCTS_TAppDecoder)

7. Obtain the YUV media files and copy to YUV_PATH

8. Run script as follows:
   
Run all test suites：

```
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type all
```

​Run test suites separately:

```
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type vbv_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type mcts_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type hdr_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type intra_period_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type width_height_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type buffered_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type run_to_run_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type qp_file_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type enc_struct_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type unpacked_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type dlf_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type sao_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type constrained_intra_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type scene_change_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type me_hme_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type asm_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type decode_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type defield_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type tile_test
python SVT-HEVC_FunctionalTests.py [Fast|Nightly|Full ] -type multi_channel_test
```

