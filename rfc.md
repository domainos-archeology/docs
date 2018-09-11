RFC = Run File Converter

This is the file format for all the standalone utilities (SAU) found in `/sau#` dirs, including `domain_os`
byte    size  name
0        4      low address at which the program is loaded
4        4      start address, which is the initial execution entry point
8        4      machine type identifier of the target machine, or 0 if the program can run on any machine type 

machine type = n in SAUn
