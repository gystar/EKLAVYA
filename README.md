# Neural Nets Can Learn Function Type Signatures From Binaries

## Authors
EKLAVYA is designed by Zheng Leong Chua, Shiqi Shen, Prateek Saxena, Zhenkai Liang.

## Dataset
The dataset available from this page is the collection of function type signatures, which include function banaries, arguments counts and types. It is a good dataset for people who want to try learning techniques or heuristive methods in binary analysis while spending less effort on collecting and preprocessing.

<!-- The whole dataset is available on [National Cybersecurity R&D Lab](https://ncl.sg/). 
 -->
 The dataset consists of three parts, which is described below:

- [binary.tar.gz](https://drive.google.com/open?id=0B2qBKMQRQLHGS1JESVQ0TlF1eWs). The compressed file saves 5168 binaries. These binaries are from 8 distinct packages: binutils, coreutils, findutils, sg3utils, utillinux, inetutils, diffutils, and usbutils.
These 5168 binaries are obtained by using two commonly used compilers: *gcc* and *clang*, with different optimization levels ranging from *O0* to *O3* for both x86 and x64.

- [pickles.tar.gz](https://drive.google.com/open?id=0B2qBKMQRQLHGdGpuTUlmMmZJYXM). The compressed file saves the assembly code of each function and the ground truth of the function arguments by parsing the DWARF debug information. 

- [clean_pickles.tar.gz](https://drive.google.com/open?id=0B2qBKMQRQLHGOFphWjkzcnV2LTQ). The compressed file saves the assembly code and the ground truth of the function arguments for sanitized functions. For this dataset, we removed the functions which are duplicates of other functions in the dataset. Given that the same piece of code compiled with different binaries will result in different offsets generated,we chose to remove all direct address used by instructions found in the function. For example, the instruction *'je 0x98'* are represented as *'je '*. After the substituion, we hash the function and remove functions with the same hashes. Other than duplicates, we also removed functions with less than four instructions as these small functions typically do not have any operation on arguments. 


###Function Representation
A function is represented as a dictionary having the following fields:

- **num_args**: Integer - Number of arguments.
- **args_type**: List - Type of each argument, as String, in the order they appear in the function declaration.
- **ret_type**: String - Type of the value returned by the function.
- **inst_strings**: List - The assembly code of each instruction composing the function, as strings.
- **inst_bytes**: List - The bytecode of each instruction composing the function, as an array of values. (Each instruction is represented by one array of bytes.)
- **boundaries**: Tuple (Integer, Integer) - The starting and ending address of the function.

**Example:**
```
FuncDict = {
	"num_args": 3,
	"args_type": ["int", "char", "struct structure1*"], 
	"ret_type": "int", 
	"inst_strings": ["mov eax, 1", "nop", "push 3"],
	"inst_bytes": [[0x01, 0x02], [0xff], [0x20, 0x30, 0x40]],
	"boundaries": (0x80010, 0x800f0)
}
```

###Binary Representation
A binary saved in **pickles.tar.gz** and **clean_pickles.tar.gz** is represented as a Dict object, having the following fields:

- **functions**: Dict - Ths dictionary contains as keys function names and as values function dictionaries which is described above. The information of dupilicate functions are removed from the dictionary.
- **structures**: Dict -The dictionary describing structures used in the binary files. Dictionary's keys are names of the structures and the values are lists containing the type of each field in the structure, in the order they are declared.
- **text_addr**: String - Address of the text section in the binary.
- **binRawBytes**: String - Raw contents of the binary file.
- **arch**: String - Binary architecture.
- **binary_filename**: String - Name of the binary elf file used to generate the binInfo.
- **function_calls**: Dict - The dictionary containing information about function calls. A caller is described using its name and an array of instruction indices, starting from 0. (If **10** is in the array, the **10th** instruction in the caller is a call to the callee.)
	- Key: callee function's name
	- Value: array of objects describing function callers

**Example:**
```
BinaryFileDict = {
    "functions": {
        "function1": funcDict1,
        "function2": funcDict2
    },
    "structures": {
        "structure1": ["int", "char [16]", "float"],
        "structure2": ["long", "long", "short"]
    },
    "text_addr": 0x800000,
    "binRawBytes": "\x00\x12\x22...",
    "arch": "i386",
    "binary_filename": "gcc-32-O1-coreutils-ls",
    "function_calls": {
         "func1": [
             {
             	 # caller's name
                 "caller": "func2",
                 # the indices of calling instructions
                 "call_instr_indices": [10, 17, 29] 
             },
             {
                 "caller": "func2",
                 "call_instr_indices": [19]
             }
        ]
    }
}
```
## Code
###Requirement

- tensorflow
- numpy

###Train Embedding Model


###Train RNN Model
Usage: 
```
python train/train.py [options] -d data_folder -o output_dir -f split_func_path -e embed_path
```

- **data_folder**: The folder saves the binary information.  
- **output_dir**: The directory is used to save the trained model & log information.
- **split_func_path**: The file saves the training & testing function names.
- **embed_path**: The file saves the embedding vector of each instruction.

Options:

- **-t**: Type of output labels. Possible value: num_args, type#0, type#1, ...
	- **num_args**: The trained model is used to predict the number of arguments for each function.
	- **type#0**: The trained model is used to predict the type of first argument for each function.
	- **type#1**: The trained model is used to predict the type of second argument for each function.
- **-dt**: Type of input data. Possible value: caller and callee.
	- **caller**: The input data is from caller.
	- **callee**: The input data is function body.
- **-pn**: Number of Processes.
- **-ed**: Dimension of embedding vector for each instruction.
- **-ml**: Maximum length of input sequences.
- **-nc**: Number of Classes.
- **-en**: Number of Epochs.
- **-s**: The frequency for saving the trained model. If the value is 100, the trained model is going to be saved every 100 batches.
- **-do**: Dropout value.
- **-nl**: Number of layers in RNN.
- **-ms**: Maximum number of model saved in the directory.
- **-b**: Batch size.
- **-p**: The frequency for showing the accuracy & cost value.

###Testing RNN Model



## Disclaimer
The code is research-quality proof of concept, and is still under development for more features and bug-fixing.

## References
[Neural Nets Can Learn Function Type Signatures From Binaries](www.google.com)

Zheng Leong Chua, Shiqi Shen, Prateek Saxena, Zhenkai Liang.

In the 26th USENIX Security Symposium (Usenix Security 2017)
