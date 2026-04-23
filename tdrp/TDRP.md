# TDRP Documentation

This Markdown file was consolidated from the original TDRP HTML documentation.

## Contents

- [TDRP Overview](#section-index)
- [How TDRP works](#section-how-it-works)
- [The TDRP command line](#section-command-line)
- [Overriding TDRP parameters on the command line.](#section-override)
- [The TDRP parameter file](#section-parameter-file)
- [The paramdef file](#section-paramdef)
- [paramdef Example](#section-paramdef-example)
- [tdrp_gen](#section-tdrp-gen)
- [TDRP API for C](#section-api-c)
- [TDRP API for C++](#section-api-cplusplus)
- [Loading Parameters into a Program](#section-load-params)
- [Saving parameters to a file](#section-saving-params)
- [Changing TDRP parameters from within a program](#section-changing-params)
- [TDRP usage](#section-usage)
- [Multiple TDRP Classes in C++](#section-mult-classes)
- [Multiple TDRP modules in C](#section-mult-modules)
- [Example in C](#section-example-c)
- [Example in C++](#section-example-cplusplus)
- [TDRP Makefiles](#section-makefiles)
- [Advantages of using TDRP](#section-advantages)
- [TDRP history](#section-history)
- [Parameter File Example](#section-parameter-file-example)
- [Old TDRP API for C](#section-api-c-old)

<a id="section-index"></a>

## TDRP Overview

# TDRP - Table-Driven Runtime Parameters

------------------------------------------------------------------------

## Summary

- Name: libtdrp, tdrp_gen
- Status: mature, used in multiple Level 3 projects
- Programming language: C/C++
- Maintainer(s): [Mike Dixon](mailto:dixon@ucar.edu)
- Part of /rap nightly build: Yes
- License:
- Dependencies:
- Location in CVS: libs/tdrp, apps/tdrp

------------------------------------------------------------------------

## TDRP - Table-Driven Runtime Parameters

TDRP consists of a small stand-alone C library and a code generation program which helps both the **user** and the **programmer** deal effectively with any parameters needed for a program to run.

TDRP is written in ANSI-C, and only an ANSI-C compiler is required to build the system which consists of the library *libtdrp* and the program *tdrp_gen*. Both C and C++ are supported explicilty.

------------------------------------------------------------------------

The **user** interacts with a TDRP-based program in two ways:

- [The command line](#section-command-line):
- [The parameter file](#section-parameter-file):

------------------------------------------------------------------------

The **programmer** goes through the following steps in setting up a TDRP-based program:

- Create the [paramdef](#section-paramdef) file:
- Run [tdrp_gen](#section-tdrp-gen) to generate the TDRP code for the program:
- Create an [override list](#section-override) from the command line args.
- [Load the parameters](#section-load-params).
- Provide the [TDRP usage](#section-usage) to the user.

------------------------------------------------------------------------

Three **applications programming interfaces** are provided:

- [C API](#section-api-c).
- [C++ API](#section-api-cplusplus).
- [Old C API](#section-api-c-old) - included for backward compatibillity.

------------------------------------------------------------------------

More **advanced programming topics** include:

- [Changing](#section-changing-params) TDRP parameters from within a program:
- [Saving](#section-saving-params) out alternative TDRP parameter sets to a file:
- Creating [multiple TDRP modules](#section-mult-modules) for a single C program.
- Creating [multiple TDRP classes](#section-mult-classes) for a single C++ program.

------------------------------------------------------------------------

**Additional topics:**

- [Advantages of this approach](#section-advantages).
- [How it works](#section-how-it-works).
- [C code example](#section-example-c).
- [C++ code example](#section-example-cplusplus).
- [A brief history.](#section-history)

------------------------------------------------------------------------

Author: Mike Dixon  
RAP, NCAR, P.O.Box 3000, Boulder, CO, 80307-3000  
*dixon@ucar.edu*  
September 1998

<a id="section-how-it-works"></a>

## How TDRP works

There are two main components of the system in memory:

- The parameter table.
- The parameter structure (C) or object (C++) in user space.

The table is a static array in the \_tdrp.c module for C, or a private array in the Params class in C++. The `init()` function loads up the table with the default values. The `read()` function then takes values from the parameter file and writes over the defaults.

Stored in the table are offsets from some reference address in memory to the address of the parameter in user space. At the end of the `read()` function the values in the table are copied across from the table into user space, using the offset to compute the user-space address.

From then on the program deals with the parameters in user space. If changes are made to parameter values this is done in user space.

Prior to a print of the parameters, the values are copied back from user space to the table via the `sync()` function.

<a id="section-command-line"></a>

## The TDRP command line

If a program complies with the current TDRP usage, the user will get the following printout when invoking the -h option on the command line:

    TDRP args: [options as below]
       [ -params path ] specify params file path.
         If not specified, the default params are loaded.   
       [ -check_params] check which params are not set
       [ -print_params [mode]] print parameters
         using following modes, default mode is 'norm'
           short:   main comments only, no help or descriptions
                    structs and arrays on a single line
           norm:    short + descriptions and help
           long:    norm  + arrays and structs expanded
           verbose: long  + private params included
           short_expand:   short with env vars expanded
           norm_expand:    norm with env vars expanded
           long_expand:    long with env vars expanded
           verbose_expand: verbose with env vars expanded
       [ -tdrp_debug] debugging prints for tdrp
       [ -tdrp_usage] print this usage

This usage message is expanded on below, with more detailed explanations for the actions. It is assumed you are working with a program named *prog_name*, and that you type in one of the following:

    prog_name

Run *prog_name* using the default parameters.

    prog_name -params param_file_name

Run *prog_name* using the parameter file *param_file_name*.

    prog_name -params param_file_name -check_params(return)

Print to the screen those parameters which are not set by the parameter file. Then exit.

    prog_name -params param_file_name -print_params

Print the parameters in the param file to stdout in standard format. If this text is captured to a file it forms a valid parameter file. Then exit.

    prog_name -print_params
    prog_name -print_params norm

Print the default parameters to stdout in standard format. Arrays and structs are printed across the screen. Then exit.

    prog_name -print_params short

Print the default parameters to stdout in short format. The comments associated with each parameter are suppressed. The commentdefs are printed. Then exit.

    prog_name -print_params long

Print the default parameters to stdout in short format. Arrays and structs are printed down the screen. Then exit.

    prog_name -print_params verbose

Print the default parameters to stdout in verbose format. This is the same as the long format except private parameters are printed as well. Note that this does not produce a valid parameter file if there are private parameters because you cannot override private params through the param file. Then exit.

    prog_name -print_params short_expand

Same as short except expand the environment variables before printing. Then exit.

    prog_name -print_params norm_expand

Same as norm except expand the environment variables before printing. Then exit.

    prog_name -print_params long_expand

Same as long except expand the environment variables before printing. Then exit.

    prog_name -print_params verbose_expand

Same as verbose except expand the environment variables before printing. Then exit.

    prog_name -tdrp_usage

Print out TDRP usage statement and then exit.

    prog_name -params param_file_name -tdrp_debug

Turn on tdrp debugging. This is sometimes useful for finding errors which the standard error messages do not describe in sufficient detail.

<a id="section-override"></a>

## Overriding TDRP parameters on the command line.

TDRP provides a mechanism to allow command line arguments to override parameter values set in the parameter file. This has the advantage of allowing the parameter-parsing logic to be applied to command line args rather than having to program separate logic to deal with the args.

The override mechanism uses a NULL-terminated array of strings, each of which is in the same format as an entry in the parameter file. The type `tdrp_override_t` is provided, along with a series of functions, to make it easy to create and use the override list.

#### Example

As an example, suppose the paramdef file contains the following:

      paramdef boolean {
        p_descr = "Option to print debugging messages";
      } debug;

The the paramfile could have the following entry:

      debug = FALSE;

Suppose a `-debug` appears on the command line. The following code fragment shows how you could set up and use the override:

      /* initialize before parsing the command line */

      tdrp_override_t override;
      TDRP_init_override(&override);

      /* in the arg parsing code */

      char tmp_str[BUFSIZ];
      int i;
      for (i =  1; i < argc; i++) {
        if (!strcmp(argv[i], "-debug")) {
          sprintf(tmp_str, "debug = TRUE;");
          TDRP_add_override(&override, tmp_str);
        }
      }

      /* loading the parameters */

      char *params_file_path;
      _tdrp_struct params;
      if (_tdrp_load_from_args(argc, argv, &params,
                   override.list, &params_file_path)) {
        fprintf(stderr, "ERROR - Problems with params file '%s'\n",
            params_file_path);
        exit(-1);
      }

      /* free up the override list */

      TDRP_free_override(&override);

Each string in the override list is appended as a line to to the lines in the param file. The entry:

      debug = FALSE;

will still exist in the param file. However, this will be superceded by the line:

      debug = TRUE;

appended from the override list. Hence the debug value will be set to TRUE.

#### Program carefully and test fully

A word of caution here. As the programmer you need to make sure that your override code is generating strings which are valid as parameter entries. Test your code thoroughly, making sure you exercise every override option.

<a id="section-parameter-file"></a>

## The TDRP parameter file

The parameter file consists of comments and parameter values. A default parameter file may be produced using the [-print_params](#section-command-line) command line argument.

An [example parameter file](#section-parameter-file-example) is provided for reference.  
This file matches the parameter definitions in the [example paramdef file](#section-paramdef-example).

#### White space

The position of white space in the file is unimportant except in strings. New-lines, tabs and spaces are ignored except within double-quoted strings.

#### Comments

You may insert comments into a parameter file using either the C /\* \*/ or C++ // style.  
Lines starting with \# are also considered comments.

#### Embedded environment variables

Environment variables may be imbedded in string values, using the syntax \$(ENV_VAR), for example \$(HOME) or \$(USER). The environment variable will be expanded before its use by the client program.

For example,

      dir = "$(HOME)/data";

might expand to

      dir = /home/dixon/data.;

#### Repeated parameters

A parameter entry may appear more than once in a parameter file. In such a case, the last entry will be used and all previous entries will be ignored. No warnings are generated for repeated entries.

#### Single-valued parameters

These appear as:

      param_name = param_value;

For example:

      age = 10;                        // integer
      debug = FALSE;                   // boolean
      file_path = "/usr/local/junk";   // string - note quotes
      speed = 21.4;                    // float
      mode = ARCHIVE;                  // enum

#### 1D arrays

These appear as:

      param_name = {value1, value2, value3, ... };

For example:

      ages = {10, 11, 12};                 // integer
      flags = {FALSE, TRUE, TRUE, FALSE};  // bool
      file_paths = {"/usr/local/junk",
                    "$(HOME)/.cshrc",
                    "/etc/printcap"},      // string
      speeds = {21.4, 10.2, 15.9};         // float
      modes = {ARCHIVE, REALTIME};         // enum

Note the use of the imbedded environment variable in file_paths. This is expanded to the environment variable value before use by the program.

If the array length is fixed, the number of elements must match the expected length.

#### 2D arrays

    These appear as:

      param_name = {{value11, value12, value13, ... },
                    {value21, value22, value23, ... },
                    {value31, value32, value33, ... },
                    {value41, value42, value43, ... },
                    ......
                    {valueN1, valueN2, valueN3, ... }
                   };

For example:

      // int
      item_count = {
        { 0, 5, 6, 11, 2, 3 },
        { 9, 8, 15, 12, 4, 4 },
        { 17, 18, 3, 7, 0, 12 },
        { 15, 10, 10, 1, 9, 1 }
      };

      // float
      rain_accumulation = {
        { 0.1, 0.6, 1.9, 12.4, 1.1 },
        { 2.3, 5.7, 12.8, 19.4, 0 },
        { 14.3, 19.3, 12.1, 3.3, 7.5 },
        { 8, 6.1, 0, 15.1, 10 }
      };

      // boolean
      compute_length = {
        { FALSE, FALSE, TRUE, TRUE, TRUE },
        { FALSE, FALSE, FALSE, FALSE, TRUE },
        { FALSE, TRUE, FALSE, TRUE, FALSE },
        { FALSE, FALSE, FALSE, TRUE, TRUE }
      };

      // string
      output_file_paths = {
        { "$(USER)/path11", "$(USER)/path21", "$(USER)/path31" },
        { "$(USER)/path12", "$(USER)/path22", "$(USER)/path32" },
        { "$(USER)/path13", "$(USER)/path23", "$(USER)/path33" },
        { "$(USER)/path14", "$(USER)/path24", "$(USER)/path34" },
        { "$(USER)/path15", "$(USER)/path25", "$(USER)/path35" },
        { "$(USER)/path16", "$(USER)/path26", "$(USER)/path36" }
      };

      // boolean
      mode = {
        { REALTIME, REALTIME, ARCHIVE, OTHER },
        { OTHER, ARCHIVE, ARCHIVE, REALTIME }
      };

If the array length is fixed, the number of elements for both dimensions must match the expected lengths.

#### Structs

Single-valued structs appear rather like 1D arrays:

     param_name = {field1, field2, field3, ...};

or

     param_name = {name1 = field1, name2 = field2, name3 = field3, ...};

The *name* descriptors are optional. They are useful in long printouts of large structs, to help the user identify which field is which.

For example:

      grid = { 100, 100, -50, -50, 2, 2.5 };

or

      grid = { nx = 100, ny = 100, -50, -50, 2, 2.5 };

1D struct arrays appeas like 2D other arrays:

      param_name = {{field1, field2, field3, ...},
                    {field1, field2, field3, ...},
                    {field1, field2, field3, ...}};

or

      param_name = {
                     {name1 = field1,
                      name2 = field2,
                      name3 =  field3, ...},
                     {name1 = field1,
                      name2 = field2,
                      name3 =  field3, ...},
                     {name1 = field1,
                      name2 = field2,
                      name3 =  field3, ...}};

Once again, the *name* descriptors are optional.

If the array length is fixed, the number of elements must match the expected length.

For example:

      surface_stations = {
        { 40.1012, -104.231, 10, ETI, TRUE},
        { 40.2109, -104.576, 10, GEONOR, FALSE},
        { 39.1379, -104.908, 3, CAMPBELL, FALSE}
      };

      data_field = {
        {
          scale = 0.5,
          bias = 1,
          nplanes = 16,
          name = "Reflectivity",
          units = "dBZ",
          origin = BOTLEFT
        }
        ,
        {
          scale = 0.6,
          bias = 1.1,
          nplanes = 17,
          name = "Velocity",
          units = "m/s",
          origin = TOPLEFT
        }
      };

<a id="section-paramdef"></a>

## The paramdef file

The paramdef - Parameter Definition - file specifies the parameters of the client program. From the programmer's perspective understanding this file is one of the keys to the TDRP system.

An example paramdef file, showing examples all of the possible parameter types, is included in [paramdef.example](#section-paramdef-example).

#### Supported parameter types

The following parameter types are supported:

- boolean
- string (or char\*)
- int
- long
- float
- double
- enum
- struct

Booleans are of type `enum tdrp_bool_t`, which can take values of `TRUE` (1) and `FALSE` (0).

Enums and structs are defined by typedefs in the paramdef file.

1D arrays are supported for all types, and 2D arrays for all types except structs. Arrays may be variable or fixed in length. 2D arrays are available to the programmer through both 1D and 2D pointers.

Struct members may be of the following types:

- boolean
- string (or char\*)
- int
- long
- float
- double
- enum

Only single-valued struct members are supported - arrays are not supported within structs, nor are structs, i.e. you cannot nest a struct within a struct.

#### White space

The position of white space in the file is unimportant except in strings. New-lines, tabs and spaces are ignored except within double-quoted strings.

#### Comments

You may insert comments into a parameter file using either the C /\* \*/ or C++ // style.  
Lines starting with \# are also considered comments.

These comments are local to the paramdef file, they are not carried forward into any code files.

#### Commentdefs

Comment definitions may be placed between the parameter definitions. Commentdefs are carried forward into the generated code files. They provide a mechanism for inserting comments into the [automatically printed parameter files](#section-command-line).

A commentdef has the following form:

    commentdef {
      p_header = "BOOLEAN PARAMETERS";
      p_text = "Testing boolean parameter behavior.";
    };

When the parameter files are automatically printed, this comment will appear as follows:

    //======================================================================
    //
    // BOOLEAN PARAMETERS.
    //
    // Testing boolean parameter behavior.
    //
    //======================================================================

This provides a way of separating the parameter files into sections delimited by the comments definded in commentdefs.

#### <span id="strings">String types</span>

String types are quoted in double quotes. Strings may be continued over multiple lines in the same style as in C and C++, i.e. the substrings appear on subsequent lines without any characters between the substrings except for white space.

For example, the following sub-strings:

     
      "This is line 1. "
      "This is line 2. "
      "this is line 3. "

will be concatenated into:

      "This is line 1. This is line 2. this is line 3."

Note that no spaces are added between the lines.

A double quote within a string may be escaped using a back-slash in the usual fashion, i.e. \\.

#### <span id="env_vars">Embedded environment variables</span>

Environment variables may be imbedded in string values, using the syntax \$(ENV_VAR), for example \$(HOME) or \$(USER). The environment variable will be expanded before its use by the client program.

For example,

      "$(HOME)/data"

might expand to

      /home/dixon/data.

#### Full parameter definition

The following example shows a full parameter definition for the integer parameter 'age':

    paramdef int {
      p_min = 0;
      p_max = 120;
      p_default = 30;
      p_private = FALSE;
      p_descr = "The age of the individual (yrs).";
      p_help = "The age is used to compute life-expectancy.";
    } age;

#### Single-valued parameters

The following example shows a full parameter definition for the integer parameter 'age':

    paramdef int {
      p_min = 0;
      p_max = 120;
      p_default = 30;
      p_private = FALSE;
      p_descr = "The age of the individual (yrs).";
      p_help = "The age is used to compute life-expectancy.";
    } age;

#### Descriptors

The 'p_xxx' in the paramdef are descriptors which add information about the parameter. All are optional. p_min and p_max are applicable only to numeric parameters.

- **p_min, p_max**:
- **p_default**:
- **p_private**:
- **p_descr**:
- **p_help**:

#### Simplest parameter definition

It is perfectly legal to reduce the paramdef to the following:

    paramdef int {
    } age;

However, this will be less useful than the more complete version. You should at least include a description, and usually a default.

#### 1D arrays

The following defines a 1D array parameter:

    paramdef int {
      p_min = 0;
      p_max = 120;
      p_default = {30, 32, 33};
      p_descr = "Our ages (yrs).";
    } our_age[];

This is a variable length array, the size is inferred from the default, in this case 3 items. If the user overrides the default in the parameter file the array will be resized to the number of values specified by the user.

A fixed length array is specified by placing an array length in the square brackets, for example:

    paramdef int {
      p_default = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
      p_descr = "Number of days in each month of the year.";
      p_help = "Testing fixed length int array.";
    } days_in_month[12];

In this case the array length is fixed. If the number of items specified does not match the fixed length an error will be generated.

Note the default - it is a list of numbers enclosed in curly braces. Items in the list are separated by commas.

#### 2D arrays

The following defines a 2D array parameter:

    paramdef int {
      p_min = 0;
      p_max = 1;
      p_default = {{0, 0, 1, 1, 1},
               {0, 0, 0, 0, 1}, 
               {0, 1, 0, 1, 0},
               {0, 0, 0, 1, 1}};
      p_descr = "Variable length 2-D array.";
    } icon[][];

This is a variable length array. Once again, putting a size in the square brackets will fix the array size. For fixed arrays you must specify a value for both dimensions.

Note that the default is a 2D nested list of numbers enclosed in curly braces. Items in the list are separated by commas.

#### Other types

The following are examples of the various simple types, not including enums and structs.

    paramdef long {
      p_default = 1;
      p_min = 0;
      p_descr = "Single long value";
      p_help = "Testing single long actions.";
    } number_of_radars;

    paramdef float {
      p_default = {101.1, 102.1, 103.1, 104.1, 105.1,
               106.1, 107.1, 108.1, 109.1, 110.1};
      p_private = FALSE;
      p_descr = "Fixed length float array.";
    } storm_volume[10];

    paramdef double {
      p_default = {{0.9, 0.9, 1.9, 1.9, 1.9, 100.3},
               {0.9, 1.9, 0.9, 1.9, 0.9, -100.1},
               {0.9, 0.9, 0.9, 1.9, 1.9, -99.9}};
      p_descr = "variable length 2-D array.";
    } length_factor[][];

    paramdef boolean {
      p_default = {TRUE, FALSE, TRUE, TRUE};
      p_private = FALSE;
      p_descr = "Bool array - variable length.";
    } allow_outliers[];

    paramdef string {
      p_default = "mcg";
      p_descr = "Input file extension";
      p_help = "Testing single-valued string parameter.";
    } input_file_ext;

    paramdef string {
      p_default = {{"$(USER)/path11", "$(USER)/path21", "$(USER)/path31"},
               {"$(USER)/path12", "$(USER)/path22", "$(USER)/path32"},
               {"$(USER)/path13", "$(USER)/path23", "$(USER)/path33"},
               {"$(USER)/path14", "$(USER)/path24", "$(USER)/path34"},
               {"$(USER)/path15", "$(USER)/path25", "$(USER)/path35"},
               {"$(USER)/path16", "$(USER)/path26", "$(USER)/path36"}};
      p_descr = "Output file paths.";
      p_help = "Testing variable length 2D array of strings."
        "Note imbedded environment variables.";
    } output_file_paths[][];

Note the [strings](#strings) in double quotes. Also note the use of the [environment variables](#env_vars) in the string values.

#### Enums

An enum requires a typedef before its use in a parameter. Here is an example:

    typedef enum {
      BOTLEFT, TOPLEFT, BOTRIGHT, TOPRIGHT
    } origin_t ;

    paramdef enum origin_t {
      p_default = {BOTLEFT, TOPLEFT};
      p_descr = "Data origin position";
      p_help = "Testing variable length enum array.";
    } data_origin[];

The typedef will appear in the generated header file for use by the client program.

Enums can specify the integer value of each element, as follows:

    typedef enum {
      ETI = 1, GEONOR = 4, CAMPBELL = 5
    } gauge_t;

Enums values are assumed to be sequential if not otherwise specified. The following typedef is equivalent to the one above.

    typedef enum {
      ETI = 1, GEONOR = 4, CAMPBELL
    } gauge_t;

Enums may be used in 2D arrays:

    typedef enum {
      REALTIME, ARCHIVE, OTHER
    } mode_t;

    paramdef enum mode_t {
      p_default = {{REALTIME, REALTIME, ARCHIVE, OTHER},
               {OTHER, ARCHIVE, ARCHIVE, REALTIME}};
      p_descr = "Testing fixed length 2-D enum array.";
    } mode[2][4];

#### Structs

Structs require a typedef prior to their use as a parameter type.

As an example of a single-valued struct:

    typedef struct {
      long nx;
      long ny;
      double minx;
      double miny;
      double dx;
      double dy;
    } grid_t;

    paramdef struct grid_t {
      p_default = {100, 100, -50.0, -50.0, dx = 2.0, 2.5};
      p_descr = "Grid parameters.";
      p_help = "Testing single-valued struct."
      "Struct Definition occurs within the paramdef.";
    } grid;

The following is an example of a 1D struct array. Note that it makes use of an enum, gauge_t, which is defined above. Enums must be defined before their use in a struct typedef.

    typedef struct {
      double lat;
      double lon;
      double wind_sensor_ht;
      gauge_t gauge_make;
      boolean has_humidity;
    } surface_station_t;

    paramdef struct surface_station_t {
      p_descr = "Surface station information.";
      p_help = "Test of variable length struct array."
      "Note that the struct is defined in a typedef before the paramdef."
      "Also, the struct includes an enum which is pre-defined. Enums included"
      "in this manned MUST be defined in a typedef.";
      p_default = {
        {40.1012, -104.2309, 10.0, ETI, TRUE},
        {40.2109, -104.5764, 10.0, GEONOR, FALSE},
        {39.1379, -104.9080, 3.00, CAMPBELL, FALSE}
      };
    } surface_stations[3];

<a id="section-paramdef-example"></a>

## paramdef Example

### Example paramdef file

    /*********************************************************
     * Example parameter definitions file
     *
     * Mike Dixon, RAP, NCAR, Boulder, CO, USA, 80307-3000
     *
     * Sept 1998
     */

    //////////////////////////////////////////////////////////

    commentdef {
      p_header = "INTEGER PARAMETERS";
      p_text = "Testing integer parameter behavior.";
    };

    paramdef int {
      p_min = 0;
      p_max = 120;
      p_default = 35;
      p_private = FALSE;
      p_descr = "Single int value";
      p_help = "Testing single int actions.";
    } your_age;

    paramdef int {
      p_min = 0;
      p_max = 120;
      p_default = {30, 31, 42, 43, 54};
      p_private = FALSE;
      p_descr = "Int array - variable length.";
      p_help = "Testing variable length int array.";
    } our_ages[];

    paramdef int {
      p_min = 0;
      p_max = 1;
      p_default = {{0, 0, 1, 1, 1},
               {0, 0, 0, 0, 1}, 
               {0, 1, 0, 1, 0},
               {0, 0, 0, 1, 1}};
      p_descr = "Variable length 2-D array.";
      p_help = "Testing variable length 2-D array.";
    } icon[][];

    //////////////////////////////////////////////////////////

    commentdef {
      p_header = "LONG INTEGER PARAMETERS";
      p_text = "Testing long integer parameter behavior.";
    };

    paramdef long {
      p_default = 1;
      p_min = 0;
      p_descr = "Single long value";
      p_help = "Testing single long actions.";
    } number_of_radars;

    paramdef long {
      p_default = {31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31};
      p_descr = "Long array - fixed length.";
      p_help = "Testing fixed length long array.";
    } days_in_month[12];

    paramdef long {
      p_min = 0;
      p_default = {{0, 5, 6, 11, 2, 3},
               {9, 8, 15, 12, 4, 4}, 
               {17, 18, 3, 7, 0, 12},
               {15, 10, 10, 1, 9, 1}};
      p_descr = "Variable fixed 2-D array.";
      p_help = "Testing fixed length 2-D array.";
    } item_count[4][6];

    //////////////////////////////////////////////////////////

    commentdef {
      p_header = "FLOAT PARAMETERS";
      p_text = "Testing float parameter behavior.";
    };

    paramdef float {
      p_default = 15.0;
      p_min = 0.0;
      p_private = FALSE;
      p_descr = "Single float value";
      p_help = "Testing single float actions.";
    } speed;

    paramdef float {
      p_default = {101.1, 102.1, 103.1, 104.1, 105.1,
               106.1, 107.1, 108.1, 109.1, 110.1};
      p_private = FALSE;
      p_descr = "Float array - fixed length.";
      p_help = "Testing fixed length float array.";
    } storm_volume[10];

    paramdef float {
      p_default = {{0.1, 0.6, 1.9, 12.4, 1.1},
               {2.3, 5.7, 12.8, 19.4, 0.0}, 
               {14.3, 19.3, 12.1, 3.3, 7.5},
               {8.0, 6.1, 0.0, 15.1, 10.0}};
      p_descr = "Variable length 2-D array.";
      p_help = "Testing variable length 2-D array.";
    } rain_accumulation[][];

    //////////////////////////////////////////////////////////

    commentdef {
      p_header = "DOUBLE PARAMETERS";
      p_text = "Testing double parameter behavior.";
    };

    paramdef double {
      p_default = 9.1e-9;
      p_descr = "Single double value";
      p_help = "Testing single double actions.";
    } mass_coefficient;

    paramdef double {
      p_default = 3.0e8;
      p_min = 2.99e8;
      p_max = 3.01e8;
      p_private = TRUE;
      p_descr = "Private double value";
      p_help = "Testing private double actions.";
    } speed_of_light;

    paramdef double {
      p_default = {1.9e8, 2.1e8, 9.7e7, 5.3e7, 1.1e9};
      p_min = 1.0;
      p_private = FALSE;
      p_descr = "Double array - variable length.";
      p_help = "Testing variable length double array.";
    } storm_mass[];

    paramdef double {
      p_default = {{0.9, 0.9, 1.9, 1.9, 1.9, 100.3},
               {0.9, 1.9, 0.9, 1.9, 0.9, -100.1},
               {0.9, 0.9, 0.9, 1.9, 1.9, -99.9}};
      p_descr = "Fixed length 2-D array.";
      p_help = "Testing fixed length 2-D array.";
    } length_factor[3][6];

    //////////////////////////////////////////////////////////

    commentdef {
      p_header = "BOOLEAN PARAMETERS";
      p_text = "Testing boolean parameter behavior.";
    };

    paramdef boolean {
      p_default = TRUE;
      p_private = FALSE;
      p_descr = "Single bool value";
      p_help = "Testing single bool actions.";
    } use_data;

    paramdef boolean {
      p_default = {TRUE, FALSE, TRUE, TRUE};
      p_private = FALSE;
      p_descr = "Bool array - variable length.";
      p_help = "Testing variable length bool array.";
    } allow_outliers[];

    paramdef boolean {
      p_default = {{FALSE, FALSE, TRUE,  TRUE,  TRUE},
               {FALSE, FALSE, FALSE, FALSE, TRUE}, 
               {FALSE, TRUE,  FALSE, TRUE,  FALSE},
               {FALSE, FALSE, FALSE, TRUE,  TRUE}};
      p_descr = "Variable length 2-D array.";
      p_help = "Testing variable length 2-D array.";
    } compute_length[][];

    paramdef boolean {
      p_default = FALSE;
      p_descr = "Option to print debugging messages";
    } debug;

    paramdef boolean {
      p_default = {TRUE, FALSE, TRUE, FALSE, TRUE, TRUE};
      p_descr = "Test boolean flags.";
    } flags[6];

    //////////////////////////////////////////////////////////

    commentdef {
      p_header = "STRING PARAMETERS";
      p_text = "Testing string parameter behavior.";
    };

    paramdef string {
      p_private = TRUE;
      p_default = "/";
      p_descr = "path delimiter";
      p_help = "Testing private string parameter.";
    } path_delim;

    paramdef string {
      p_default = "mcg";
      p_descr = "Input file extension";
      p_help = "Testing single-valued string parameter.";
    } input_file_ext;

    paramdef string {
      p_default = {"$(HOME)/path1", "$(HOME)/paths", "$(HOME)/path3"};
      p_descr = "Input file paths";
      p_help = "Testing variable length array of strings. "
        "Note imbedded environment variables.";
    } input_file_paths[];

    paramdef string {
      p_default = {{"$(USER)/path11", "$(USER)/path21", "$(USER)/path31"},
               {"$(USER)/path12", "$(USER)/path22", "$(USER)/path32"},
               {"$(USER)/path13", "$(USER)/path23", "$(USER)/path33"},
               {"$(USER)/path14", "$(USER)/path24", "$(USER)/path34"},
               {"$(USER)/path15", "$(USER)/path25", "$(USER)/path35"},
               {"$(USER)/path16", "$(USER)/path26", "$(USER)/path36"}};
      p_descr = "Output file paths.";
      p_help = "Testing variable length 2D array of strings."
        "Note imbedded environment variables.";
    } output_file_paths[][];

    paramdef string {
      p_default = "$(HOME)/input_dir";
      p_descr = "Input directory";
      p_help = "Path of input directory - realtime mode only"
        "Note imbedded environment variables.";
    } input_dir;

    //////////////////////////////////////////////////////////

    commentdef {
      p_header = "ENUM PARAMETERS";
      p_text = "Testing enum parameter behavior.";
    };

    typedef enum {
      ETI = 1, GEONOR = 2, CAMPBELL = 3
    } gauge_t;

    typedef enum {
      BOTLEFT, TOPLEFT, BOTRIGHT, TOPRIGHT
    } origin_t ;

    paramdef enum origin_t {
      p_default = {BOTLEFT, TOPLEFT};
      p_descr = "Data origin position";
      p_help = "Testing variable length enum array.";
    } data_origin[];

    paramdef enum mode_t {
      p_options = {REALTIME, ARCHIVE, OTHER};
      p_default = {{REALTIME, REALTIME, ARCHIVE, OTHER},
               {OTHER, ARCHIVE, ARCHIVE, REALTIME}};
      p_descr = "Testing 2-D enum array.";
      p_help = "The options for this enum are defined in the paramdef "
      "instead of in a typedef.";
    } mode[2][4];

    //////////////////////////////////////////////////////////

    commentdef {
      p_header = "STRUCT PARAMETERS";
      p_text = "Testing struct parameter behavior.";
    };

    paramdef struct grid {
      p_descr = "Grid parameters.";
      p_help = "Testing single-valued struct."
      "Struct Definition occurs within the paramdef.";
      p_field_type = {long, long, double, double, double, double};
      p_field_name = {nx, ny, minx, miny, dx, dy};
      p_default = {100, 100, -50.0, -50.0, dx = 2.0, 2.5};
    } grid;

    typedef struct {
      double lat;
      double lon;
      double wind_sensor_ht;
      gauge_t gauge_make;
      boolean has_humidity;
    } surface_station_t;

    paramdef struct surface_station_t {
      p_descr = "Surface station information.";
      p_help = "Test of variable length struct array."
      "Note that the struct is defined in a typedef before the paramdef."
      "Also, the struct includes an enum which is pre-defined. Enums included"
      "in this manned MUST be defined in a typedef.";
      p_default = {
        {40.1012, -104.2309, 10.0, ETI, TRUE},
        {40.2109, -104.5764, 10.0, GEONOR, FALSE},
        {39.1379, -104.9080, 3.00, CAMPBELL, FALSE}
      };
    } surface_stations[3];

    typedef struct {
      double scale;
      double bias;
      long nplanes;
      string name;
      string units;
      origin_t origin;
    } data_field_t;

    paramdef struct data_field_t {
      p_descr = "Data field parameters";
      p_help = "Test of fixed-length struct array.";
      p_default = {{0.5, bias = 1.0, 16, "Reflectivity", "dBZ", BOTLEFT},
               {0.6, 1.1, 17, "Velocity", units = "m/s", TOPLEFT}};
    } data_field[];

<a id="section-tdrp-gen"></a>

## tdrp_gen

tdrp_gen parses a TDRP paramdef file and then generates either C or C++ headers and code for reading in the parameter file at run-time and making the parameters visible to the client program. Source code for this program exists under apps/tdrp

The usage for tdrp_gen is given below:

    Usage:
      tdrp_gen [moduleName] -f paramdef_path
               [-h] [-c++] [-class className] [-prog progName] [-debug]

    where:
      [moduleName] in C mode all externals are prepended with this name.
        moduleName must be first arg if it is specified.
        If first arg begins with -, moduleName is set
        to empty string.
      [-f paramdef_path] parameter definition file path.
                         This arg is REQUIRED.
      [-h] gives usage.
      [-c++] C++ mode - generates .hh and .cc class files.
      [-class className] In C++ mode, set the name of the params class.
                         Default is 'Params'.
      [-prog progName] Program name for documenting code files.
      [-debug] print debug messages.

    NOTES: TDRP - Table Driven Runtime Parameters.
      tdrp_gen performs code generation.
      tdrp_gen will generate two files, one header and one for code.
      In C mode, the default, it will generate the files:
        moduleName_tdrp.h and moduleName_tdrp.c.
      If moduleName is left out of the command line, the files will be:
        _tdrp.h and _tdrp.c.
      In C++ mode, it will generate the files:
        className.hh and className.cc.
      If the -class arg is not given, the files will be:
        Params.hh and Params.cc.

**tdrp_gen supports the following:**

- generation of C code for a single TDRP module per program.
- generation of C code for multiple TDRP modules per program.
- generation of C++ code for a single TDRP class per program.
- generation of C++ code for multiple TDRP classes per program.

**Generation of C code for a single TDRP module per program.**

If you run:

      tdrp_gen -f paramdef_path -prog name

tdrp_gen will generate two files, `_tdrp.h` and `_tdrp.c`. This is the normal case, in which a single TDRP module is used with a program. The module name is omitted, and the generated files (and the routines within them) start with ***\_tdrp**.*

**Generation of C code for multiple TDRP modules per program.**

If you run:

      tdrp_gen moduleName -f paramdef_path -prog name

tdrp_gen will generate two files, `moduleName_tdrp.h` and `moduleName_tdrp.c`. The the generated files (and the routines within them) start with ***moduleName_tdrp**. This allows the programmer to set up multiple TDRP modules. The moduleName provides the unique name-space for each module.*

**Generation of C++ code for a single TDRP class per program.**

If you run:

      tdrp_gen -f paramdef_path -C++ -prog name

tdrp_gen will generate two files, `Params.hh` and `Params.cc`. This is the normal case, in which a single TDRP class is used with a program. The class name is omitted, so the default value of ***Params** is used.*

**Generation of C++ code for multiple TDRP classes per program.**

If you run:

      tdrp_gen -f paramdef_path -C++ -class className -prog name

tdrp_gen will generate two files, `className.hh` and `className.cc`. Running tdrp_gen multiple times with different paramdef file and classNames will produce multiple classes.

<a id="section-api-c"></a>

## TDRP API for C

When tdrp_gen is run on a paramdef file in C mode, it produces two files:

      moduleName_tdrp.h
      moduleName_tdrp.c

In normal cases, in which only one parameter file is required for a program, the module is set to the empty string. Then the generated files are:

      _tdrp.h
      _tdrp.c

These two files contain most of the API for the simple case. Some additional routines are defined in *tdrp/tdrp.h*. The following are the relevant API routines from these sources. If a moduleName is used, all of the \_tdrp identifiers will be prepended with moduleName.

    /*************************************************************
     * _tdrp_load_from_args()
     *
     * Loads up TDRP using the command line args.
     *
     * Check TDRP_usage() for command line actions associated with
     * this function.
     *
     *   argc, argv: command line args
     *
     *   _tdrp_struct *params: loads up this struct
     * 
     *   char **override_list: A null-terminated list of overrides
     *     to the parameter file.
     *     An override string has exactly the format of an entry
     *     in the parameter file itself.
     *
     *   char **params_path_p: if non-NULL, this is set to point to
     *                         the path of the params file used.
     *
     *  Returns 0 on success, -1 on failure.
     */

    extern int _tdrp_load_from_args(int argc, char **argv,
                                    _tdrp_struct *params,
                                    char **override_list,
                                    char **params_path_p);

    /*************************************************************
     * _tdrp_load()
     *
     * Loads up TDRP for a given module.
     *
     * This version of load gives the programmer the option to load
     * up more than one module for a single application. It is a
     * lower-level routine than _tdrp_load_from_args,
     * and hence more flexible, but the programmer must do more work.
     *
     *   char *param_file_path: the parameter file to be read in.
     *
     *   _tdrp_struct *params: loads up this struct
     *
     *   char **override_list: A null-terminated list of overrides
     *     to the parameter file.
     *     An override string has exactly the format of an entry
     *     in the parameter file itself.
     *
     *  expand_env: flag to control environment variable
     *                expansion during tokenization.
     *              If TRUE, environment expansion is set on.
     *              If FALSE, environment expansion is set off.
     *
     *  Returns 0 on success, -1 on failure.
     */

    extern int _tdrp_load(char *param_file_path,
                          _tdrp_struct *params,
                          char **override_list,
                          int expand_env, int debug);

    /*************************************************************
     * _tdrp_load_defaults()
     *
     * Loads up defaults for a given module.
     *
     * See _tdrp_load() for more details.
     *
     *  Returns 0 on success, -1 on failure.
     */

    extern int _tdrp_load_defaults(_tdrp_struct *params,
                                   int expand_env);

    /*************************************************************
     * _tdrp_sync()
     *
     * Syncs the user struct data back into the parameter table,
     * in preparation for printing.
     */

    extern void _tdrp_sync(void);

    /*************************************************************
     * _tdrp_print()
     * 
     * Print params file
     *
     * The modes supported are:
     *
     *   PRINT_SHORT:   main comments only, no help or descriptions
     *                  structs and arrays on a single line
     *   PRINT_NORM:    short + descriptions and help
     *   PRINT_LONG:    norm  + arrays and structs expanded
     *   PRINT_VERBOSE: long  + private params included
     */

    extern void _tdrp_print(FILE *out, tdrp_print_mode_t mode);

    /*************************************************************
     * _tdrp_free_all()
     *
     * Frees up all TDRP dynamic memory.
     */

    extern void _tdrp_free_all(void);

    /*************************************************************
     * _tdrp_check_all_set()
     *
     * Return TRUE if all set, FALSE if not.
     *
     * If out is non-NULL, prints out warning messages for those
     * parameters which are not set.
     */

    extern int _tdrp_check_all_set(FILE *out);

    /*************************************************************
     * _tdrp_array_realloc()
     *
     * Realloc 1D array.
     *
     * If size is increased, the values from the last array entry is
     * copied into the new space.
     *
     * Returns 0 on success, -1 on error.
     */

    extern int _tdrp_array_realloc(char *param_name,
                                   int new_array_n);

    /*************************************************************
     * _tdrp_array2D_realloc()
     *
     * Realloc 2D array.
     *
     * If size is increased, the values from the last array entry is
     * copied into the new space.
     *
     * Returns 0 on success, -1 on error.
     */

    extern int _tdrp_array2D_realloc(char *param_name,
                                     int new_array_n1,
                                     int new_array_n2);

    /*************************************************************
     * _tdrp_table()
     *
     * Returns pointer to static Table for this module.
     */

    extern TDRPtable *_tdrp_table(void);

    /*************************************************************
     * _tdrp_init()
     *
     * Module table initialization function.
     *
     *
     * Returns pointer to static Table for this module.
     */

    extern TDRPtable *_tdrp_init(_tdrp_struct *params);

    /**********************
     * TDRP_init_override()
     *
     * Initialize the override list
     */

    extern void TDRP_init_override(tdrp_override_t *override);

    /*********************
     * TDRP_add_override()
     *
     * Add a string to the override list
     */

    extern void TDRP_add_override(tdrp_override_t *override, char *override_str);

    /**********************
     * TDRP_free_override()
     *
     * Free up the override list
     */

    extern void TDRP_free_override(tdrp_override_t *override);

    /***********************************************************
     * TDRP_str_replace()
     *
     * Replace a string.
     */

    extern void TDRP_str_replace(char **s1, char *s2);

    /*********************************************************
     * TDRP_usage()
     *
     * Prints out usage message for TDRP args as passed in to
     * TDRP_load_from_args()
     */

    extern void TDRP_usage(FILE *out);

<a id="section-api-cplusplus"></a>

## TDRP API for C++

When tdrp_gen is run on a paramdef file in C++ mode, it produces two files:

      className.hh
      className.cc

In normal cases, in which only one parameter file is required for a program, the default class 'Params' is used. Then the generated files are:

      Params.hh
      Params.cc

These two files contain most of the API for the simple case. Some additional routines are defined in *tdrp/tdrp.h*. The following are the relevant API routines from these sources.

    //////////////////////////////////////////////////////////////
    // Class definition
    //////////////////////////////////////////////////////////////

    class Params {

    public:

      // Member functions

      Params();

      ~Params();

      int loadFromArgs(int argc, char **argv,
                       char **override_list,
                       char **params_path_p);

      int load(char *param_file_path,
               char **override_list,
               int expand_env, int debug);

      int loadDefaults(int expand_env);

      void sync();

      void print(FILE *out, tdrp_print_mode_t mode);

      int checkAllSet(FILE *out);

      int arrayRealloc(char *param_name,
                       int new_array_n);

      int array2DRealloc(char *param_name,
                         int new_array_n1,
                         int new_array_n2);

      void freeAll(void);

      // Data members

      etc. etc.

    private:

      etc. etc.

    };

    //////////////////////////////////////////////////////////////
    // API details
    //////////////////////////////////////////////////////////////

    //////////////////////////////////////////////////////////////
    // loadFromArgs()
    //
    // Loads up TDRP using the command line args.
    //
    // Check TDRP_usage() for command line actions associated with
    // this function.
    //
    //   argc, argv: command line args
    //
    //   char **override_list: A null-terminated list of overrides
    //     to the parameter file.
    //     An override string has exactly the format of an entry
    //     in the parameter file itself.
    //
    //   char **params_path_p: if non-NULL, this is set to point to
    //                         the path of the params file used.
    //
    //  Returns 0 on success, -1 on failure.
    //

    int Params::loadFromArgs(int argc, char **argv,
                             char **override_list,
                             char **params_path_p);

    //////////////////////////////////////////////////////////////
    // load()
    //
    // Loads up TDRP for a given class.
    //
    // This version of load gives the programmer the option to load
    // up more than one class for a single application. It is a
    // lower-level routine than loadFromArgs,
    // and hence more flexible, but the programmer must do more work.
    //
    //   char *param_file_path: the parameter file to be read in.
    //
    //   char **override_list: A null-terminated list of overrides
    //     to the parameter file.
    //     An override string has exactly the format of an entry
    //     in the parameter file itself.
    //
    //  expand_env: flag to control environment variable
    //                expansion during tokenization.
    //              If TRUE, environment expansion is set on.
    //              If FALSE, environment expansion is set off.
    //
    //  Returns 0 on success, -1 on failure.
    //

    int Params::load(char *param_file_path,
                     char **override_list,
                     int expand_env, int debug);

    //////////////////////////////////////////////////////////////
    // loadDefaults()
    //
    // Loads up default params for a given class.
    //
    // See load() for more detailed info.
    //
    //  Returns 0 on success, -1 on failure.
    //

    int Params::loadDefaults(int expand_env);

    //////////////////////////////////////////////////////////////
    // sync()
    //
    // Syncs the user struct data back into the parameter table,
    // in preparation for printing.
    //

    void Params::sync(void);

    //////////////////////////////////////////////////////////////
    // print()
    // 
    // Print params file
    //
    // The modes supported are:
    //
    //   PRINT_SHORT:   main comments only, no help or descriptions
    //                  structs and arrays on a single line
    //   PRINT_NORM:    short + descriptions and help
    //   PRINT_LONG:    norm  + arrays and structs expanded
    //   PRINT_VERBOSE: long  + private params included
    //

    void Params::print(FILE *out, tdrp_print_mode_t mode);

    //////////////////////////////////////////////////////////////
    // checkAllSet()
    //
    // Return TRUE if all set, FALSE if not.
    //
    // If out is non-NULL, prints out warning messages for those
    // parameters which are not set.
    //

    int Params::checkAllSet(FILE *out);

    //////////////////////////////////////////////////////////////
    // freeAll()
    //
    // Frees up all TDRP dynamic memory.
    //

    void Params::freeAll(void);

    //////////////////////////////////////////////////////////////
    // arrayRealloc()
    //
    // Realloc 1D array.
    //
    // If size is increased, the values from the last array entry is
    // copied into the new space.
    //
    // Returns 0 on success, -1 on error.
    //

    int Params::arrayRealloc(char *param_name, int new_array_n);

    //////////////////////////////////////////////////////////////
    // array2DRealloc()
    //
    // Realloc 2D array.
    //
    // If size is increased, the values from the last array entry is
    // copied into the new space.
    //
    // Returns 0 on success, -1 on error.
    //

    int Params::array2DRealloc(char *param_name,
                               int new_array_n1,
                               int new_array_n2);

    /**********************
     * TDRP_init_override()
     *
     * Initialize the override list
     */

    extern void TDRP_init_override(tdrp_override_t *override);

    /*********************
     * TDRP_add_override()
     *
     * Add a string to the override list
     */

    extern void TDRP_add_override(tdrp_override_t *override, char *override_str);

    /**********************
     * TDRP_free_override()
     *
     * Free up the override list
     */

    extern void TDRP_free_override(tdrp_override_t *override);

    /***********************************************************
     * TDRP_str_replace()
     *
     * Replace a string.
     */

    extern void TDRP_str_replace(char **s1, char *s2);

    /*********************************************************
     * TDRP_usage()
     *
     * Prints out usage message for TDRP args as passed in to
     * TDRP_load_from_args()
     */

    extern void TDRP_usage(FILE *out);

<a id="section-load-params"></a>

## Loading Parameters into a Program

### Loading HTML parameters into a program

The simplest way to load the parameters is to use one of the load_from_args() functions. These functions parse the command line for any [command line arguments](#section-command-line) specific to TDRP and act on them. These functions are suitable only in the standard case of a single TDRP module or class per program. For multiple modules or classes, refer to the API pages.

**Using [C API](#section-api-c):**

\_tdrp_load_from_args()

The following code fragment shows how to load the parameters into the `params` struct. It is assumed that the [override](#section-override) list has already been initialized and loaded.

      char *params_file_path = NULL;
      tdrp_override_t override;
      _tdrp_struct params;   /* parameter struct */
      if (_tdrp_load_from_args(argc, argv, &params,
                   override.list, &params_file_path)) {
        fprintf(stderr, "ERROR - Problems with params file '%s'\n",
            params_file_path);
        return(-1);
      }

After loading, the parameters are available through the struct `params`.

**Using [C++ API](#section-api-cplusplus):**

loadFromArgs()

The following code fragment shows how to load the parameters into the `Params` class. It is assumed that the [override](#section-override) list has already been initialized and loaded.

      Params *params = new Params();
      char *paramsPath = "unknown";
      if (params->loadFromArgs(argc, argv,
                               _args->override.list,
                               &_paramsPath)) {
        fprintf(stderr, "ERROR: problems with TDRP parameters\n");
        return;
      }

After loading, the parameters are available through the object `params`.

**Using [Old C API](#section-api-c-old):**

TDRP_load_from_args()

The following code fragment shows how to load the parameters into the `params` struct. It is assumed that the [override](#section-override) list has already been initialized and loaded.

      char *params_file_path = NULL;
      tdrp_override_t override;
      TDRPtable *table;                  /* TDRP parsing table */
      tdrp_example_tdrp_struct params;   /* parameter struct */

      table = tdrp_example_tdrp_init(&params);
      if (TDRP_load_from_args(argc, argv,
                  table, &params,
                  override.list, &params_file_path)) {
        fprintf(stderr, "ERROR - Problems with params file '%s'\n",
            params_file_path);
        return(-1);
      }

After loading, the parameters are available through the struct `params`.

<a id="section-saving-params"></a>

## Saving parameters to a file

Some programs, especially those driven by a GUI, will [change TDRP parameters](#section-changing-params) from within the program.

If you have changed parameters in the program, you may print the amended values out to a file for later use. To do this, use the `sync()` function to copy the the changes out to the parameter table and the `print()` function to print to a file.

Example using the [C API](#section-api-c):

      /*
       * copy params back to table
       */

      _tdrp_sync();

      /*
       * print out to params.out
       */

      if ((out = fopen("params.out", "w")) == NULL) {
        fprintf(stderr, "ERROR\n");
        fprintf(stderr,
            "Cannot open file params.out for writing table.\n");
        return (-1);
      }
      _tdrp_print(out, PRINT_SHORT);
      fclose(out);

Example using the [C++ API](#section-api-cplusplus):

      Params *params;

      // copy params back to table

      _params->sync();

      // print out to params.out

      if ((out = fopen("params.out", "w")) == NULL) {
        fprintf(stderr, "ERROR\n");
        fprintf(stderr,
            "Cannot open file params.out for writing table.\n");
        return (-1);
      }
      _params->print(out, PRINT_SHORT);
      fclose(out);

<a id="section-changing-params"></a>

## Changing TDRP parameters from within a program

#### Changing non-string parameters

You may set any of the following parameter types directly from within a program:

- boolean
- int
- long
- float
- double
- enum

Also, any struct fields of these types may be set directly.

#### Changing string parameters

To change the vale of a string parameter, use TDRP_str_replace().

For example, if `params` points to the struct of parameters defined in [paramdef.example](#section-paramdef-example), you could do:

      TDRP_str_replace(&params.input_file_ext, "dob");
      TDRP_str_replace(&params._input_file_paths[1],
               "New_file_path");
      TDRP_str_replace(&params._data_field[1].name, "Spectral width");
      TDRP_str_replace(&params._data_field[2].name, "Signal-to-noise");
      TDRP_str_replace(&params._data_field[2].units, "dB");

#### Changing array sizes

You should not realloc TDRP array pointers directly. Rather, you should use the realloc functions provided in the API:

Using [C API](#section-api-c):  
\_tdrp_array_realloc()  
\_tdrp_array2D_realloc()

Using [C++ API](#section-api-cplusplus):  
Params::arrayRealloc()  
Params::array2DRealloc()

Using [Old C API](#section-api-c-old):  
TDRP_array_realloc()  
TDRP_array2D_realloc()

For example, if `params` points to the struct of parameters defined in [paramdef.example](#section-paramdef-example), you could, using the C API, do:

      _tdrp_array_realloc("our_ages", 7);
      _tdrp_array_realloc("data_field", 3);
      _tdrp_array2D_realloc("rain_accumulation", 6, 8);
      _tdrp_array2D_realloc("output_file_paths", 4, 3);

After changing parameters inside a program, it is possible to [save that parameter state](#section-saving-params) out to a file.

<a id="section-usage"></a>

## TDRP usage

The TDRP usage is printed out for the user if the [command line option](#section-command-line) '-tdrp_usage' is invoked.

However, it is also conventional for the programmer to provide a complete usage for the program via the '-h' command line arg. In this case, the program should print out its specific usage, and then follow up with the TDRP usage.

The following code shows an example of a function which does this:

    static void usage(char *prog_name, FILE *out)
    {
      fprintf(out, "%s%s%s%s",
          "Usage: ", prog_name, " [options as below]\n",
          "   [ -h] produce this list.\n"
          "   [ -debug ] print debug messages\n"
          ..........);
      TDRP_usage(out);
    }

The following code fragment demonstrates how it may be invoked:

      for (i =  1; i < argc; i++) {
        if (!strcmp(argv[i], "-h")) {
          usage(prog_name, stdout);
          exit(0);
        } else if .....

        }
      } /* i */

<a id="section-mult-classes"></a>

## Multiple TDRP Classes in C++

### Multiple TDRP classs in C++

Complicated applications may require more than one TDRP class, although this is rare in practice. TDRP supports multiple classes.

Suppose you need two classes, Class1 and Class2. You would then create two paramdef files, say paramdef.Class1 and paramdef.Class2. The [Makefile](#section-makefiles) can easily be set up to generate two code classes.

The following files would be generated:

      Class1.hh
      Class1.cc
      Class2.hh
      Class2.cc

Each class has its own user interface. Instead of using `_tdrp_load_from_args()`, you would use the functions:

      Class1::load()
      Class2::load()

to load up the parameters for each class.

Since the load() functions operate at a lower level than \_load_from_args(), you will need to write code to parse the command line and respond to any TDRP arguments.

<a id="section-mult-modules"></a>

## Multiple TDRP modules in C

Complicated applications may require more than one TDRP module, although this is rare in practice. TDRP supports multiple modules.

Suppose you need two modules, mod1 and mod2. You would then create two paramdef files, say paramdef.mod1 and paramdef.mod2. The [Makefile](#section-makefiles) can easily be set up to generate two code modules.

The following files would be generated:

      mod1_tdrp.h
      mod1_tdrp.c
      mod2_tdrp.h
      mod2_tdrp.c

Each module has its own user interface. Instead of using `_tdrp_load_from_args()`, you would declare the following variables:

      mod1_tdrp_struct params1;
      mod2_tdrp_struct params2;

Using `_tdrp_load_from_args()`, you would use the functions:

      mod1_tdrp_load()
      mod2_tdrp_load()

to load up the parameters for each module.

Since the load() functions operate at a lower level than \_load_from_args(), you will need to write code to parse the command line and respond to any TDRP arguments.

<a id="section-example-c"></a>

## Example in C

#### The area_compute application

The program area_compute is a small application which demonstrates some of the TDRP functionality.

area_compute computes the area of a shape of a given size. The size is set in the parameter file. The user may choose one of three shapes, a square, circle or equilateral triangle. The area is computed and the result is written to a file. The file path is also set in the parameter file.

All of the parameters may be overridden by command line arguments.

------------------------------------------------------------------------

#### Paramdef file - paramdef.area_compute

    /*********************************************************
     * parameter definitions for area_compute
     *
     * Mike Dixon, RAP, NCAR, Boulder, CO, USA, 80307-3000
     *
     * Sept 1998
     */

    /*
     * area_compute is a small TDRP demonstration program.
     *
     * The program allows the user to compute the area of
     * a geometric shape of a given size.
     *
     * The result is printed to a file.
     */

    //////////////////////////////////////////////////////////

    paramdef boolean {
      p_default = FALSE;
      p_descr = "Option to print debugging messages";
    } debug;

    typedef enum {
      SQUARE, CIRCLE, EQ_TRIANGLE
    } shape_t ;

    paramdef enum shape_t {
      p_default = SQUARE;
      p_descr = "Shape type.";
      p_help = "The program will compute the area of a square,"
      "circle or equilateral triangle.";
    } shape;

    paramdef float {
      p_default = 1.0;
      p_min = 0.0;
      p_descr = "Size of the shape.";
    } size;

    paramdef string {
      p_default = "./area_compute.out";
      p_descr = "The path of the file to which the output is written.";
      p_help = "The directory which contains this path must exist.";
    } output_path;

------------------------------------------------------------------------

#### Parameter file - area_compute.params

The following was generated by running:  

      area_compute -print_params > file

    /**********************************************************************
     * TDRP params for area_compute
     **********************************************************************/

    ///////////// debug ///////////////////////////////////
    //
    // Option to print debugging messages.
    // Type: boolean
    //

    debug = FALSE;

    ///////////// shape ///////////////////////////////////
    //
    // Shape type.
    // The program will compute the area of a square, circle or equilateral 
    //   triangle.
    //
    // Type: enum
    // Options:
    //   SQUARE, CIRCLE, EQ_TRIANGLE
    //
    //

    shape = SQUARE;

    ///////////// size ////////////////////////////////////
    //
    // Size of the shape.
    // Minimum val: 0
    // Type: float
    //

    size = 1;

    ///////////// output_path /////////////////////////////
    //
    // The path of the file to which the output is written.
    // The directory which contains this path must exist.
    // Type: string
    //

    output_path = "./area_compute.out";

------------------------------------------------------------------------

#### Main header file - area_compute.h

    #include 
    #include 
    #include "_tdrp.h"

    /*
     * function prototypes
     */

    extern void parse_args(int argc,
                   char **argv,
                   char *prog_name,
                   tdrp_override_t *override);

    extern int write_result(_tdrp_struct *params, double area);

------------------------------------------------------------------------

#### Main c file - area_compute.c

    #include "area_compute.h"

    int main(int argc, char **argv)

    {
      
      /*
       * basic declarations
       */

      char *prog_name;
      char *params_file_path = NULL;
      tdrp_override_t override;
      _tdrp_struct params;   /* parameter struct */
      double area;

      /*
       * set program name
       */
      
      prog_name = strrchr(argv[0], '/');
      if (prog_name == NULL) {
        prog_name = argv[0];
      }

      /*
       * initialize the override list
       */
      
      TDRP_init_override(&override);

      /*
       * parse command line arguments
       */
      
      parse_args(argc, argv, prog_name, &override);
      
      /*
       * load up parameters
       */
      
      if (_tdrp_load_from_args(argc, argv, &params,
                   override.list, &params_file_path)) {
        fprintf(stderr, "ERROR - %s:main\n", prog_name);
        if (params_file_path) {
          fprintf(stderr, "Problems with params file '%s'\n",
              params_file_path);
        }
        exit(-1);
      }

      /*
       * free up override list
       */
      
      TDRP_free_override(&override);

      /*
       * compute area
       */

      switch (params.shape) {

      case SQUARE:
        area = params.size * params.size;
        break;

      case CIRCLE:
        area = params.size * params.size * (3.14159 / 4.0);
        break;

      case EQ_TRIANGLE:
        area = params.size * params.size * (0.866 / 2.0);
        break;

      } /* switch */

      /*
       * debug message
       */

      if (params.debug) {
        fprintf(stderr, "Size is: %g\n", params.size);
        switch (params.shape) {
        case SQUARE:
          fprintf(stderr, "Shape is SQUARE\n");
          break;
        case CIRCLE:
          fprintf(stderr, "Shape is CIRCLE\n");
          break;
        case EQ_TRIANGLE:
          fprintf(stderr, "Shape is EQ_TRIANGLE\n");
          break;
        } /* switch */
        fprintf(stderr, "Area is: %g\n", area);
      }
      
      /*
       * write out the result
       */
      
      write_result(&params, area);

      /*
       * Free up
       */

      _tdrp_free_all();

      return(0);

    }

------------------------------------------------------------------------

#### Parsing the command line - parse_args.c

    #include "area_compute.h"

    static void usage(char *prog_name, FILE *out)

    {

      fprintf(out, "%s%s%s%s",
          "Usage: ", prog_name, " [options as below]\n",
          "   [ -h] produce this list.\n"
          "   [ -debug ] print debug messages\n"
          "   [ -output ?] set output_path\n"
          "   [ -size ?] set shape size\n"
          "   [ -shape ?] set shape type\n"
          "      options are SQUARE, CIRCLE and EQ_TRIANGLE\n");
      TDRP_usage(out);
      
    }

    void parse_args(int argc,
            char **argv,
            char *prog_name,
                    tdrp_override_t *override)

    {

      int error_flag = 0;
      int i;
      char tmp_str[BUFSIZ];
      
      /*
       * look for command options
       */

      for (i =  1; i < argc; i++) {

        if (!strcmp(argv[i], "-h")) {
      
          usage(prog_name, stdout);
          exit(0);
          
        } else if (!strcmp(argv[i], "-debug")) {
          
          sprintf(tmp_str, "debug = TRUE;");
          TDRP_add_override(override, tmp_str);
          
        } else if (!strcmp(argv[i], "-size")) {
          
          if (i < argc - 1) {
            sprintf(tmp_str, "size = %s;", argv[++i]);
        TDRP_add_override(override, tmp_str);
          } else {
        error_flag = TRUE;
          }
        
        } else if (!strcmp(argv[i], "-shape")) {
          
          if (i < argc - 1) {
            sprintf(tmp_str, "shape = %s;", argv[++i]);
        TDRP_add_override(override, tmp_str);
          } else {
        error_flag = TRUE;
          }
        
        } else if (!strcmp(argv[i], "-output")) {
          
          if (i < argc - 1) {
            sprintf(tmp_str, "output_path = %s;", argv[++i]);
        TDRP_add_override(override, tmp_str);
          } else {
        error_flag = TRUE;
          }
        
        } /* if */
        
      } /* i */
      
      /*
       * print message if error flag set
       */

      if(error_flag) {
        usage(prog_name, stderr);
        exit(-1);
      }

      return;

    }

------------------------------------------------------------------------

#### Writing out the results - write_result.c

    #include "area_compute.h"

    int write_result(_tdrp_struct *params, double area)

    {

      FILE *out;

      if ((out = fopen(params->output_path, "w")) == NULL) {
        perror(params->output_path);
        return (-1);
      }

      fprintf(out, "Size is: %g\n", params->size);
      switch (params->shape) {
      case SQUARE:
        fprintf(out, "Shape is SQUARE\n");
        break;
      case CIRCLE:
        fprintf(out, "Shape is CIRCLE\n");
        break;
      case EQ_TRIANGLE:
        fprintf(out, "Shape is EQ_TRIANGLE\n");
        break;
      } /* switch */
      fprintf(out, "Area is: %g\n", area);

      fclose(out);

      return (0);

    }

------------------------------------------------------------------------

#### The Makefile

    PROGNAME = area_compute

    CC = gcc
    RM = /bin/rm -f
    INCLUDES = -I../include
    CFLAGS = -g
    LDFLAGS = -L.. -ltdrp

    SRCS = \
        _tdrp.c \
        write_result.c \
        parse_args.c \
        area_compute.c

    OBJS = $(SRCS:.c=.o)

    # link

    $(PROGNAME): $(OBJS)
        $(CC) -o $(PROGNAME) $(OBJS) $(LDFLAGS)

    # tdrp

    _tdrp.c: paramdef.$(PROGNAME)
        tdrp_gen -f paramdef.$(PROGNAME) -prog $(PROGNAME)

    clean_tdrp:
        $(RM) _tdrp.h _tdrp.c

    clean:
        $(RM) core a.out
        $(RM) *.i *.o  *.ln *~

    clean_bin:
        $(RM) $(PROGNAME)

    clean_all: clean clean_bin clean_tdrp

    # suffix rules

    .SUFFIXES: .c .o

    .c.o:
        $(CC) -c $(CFLAGS) $(INCLUDES) $<

------------------------------------------------------------------------

#### The TDRP header file - \_tdrp.h

    /*******************************************
     * _tdrp.h
     *
     * TDRP header file.
     *
     * Code for program area_compute
     *
     * This header file has been automatically
     * generated by TDRP, do not modify.
     *
     *******************************************/

    #ifndef __tdrp_h
    #define __tdrp_h

    #ifdef __cplusplus
    extern "C" {
    #endif

    #include 

    /*
     * typedefs
     */

    typedef enum {
      SQUARE = 0,
      CIRCLE = 1,
      EQ_TRIANGLE = 2
    } _shape_t;

    /*
     * typedef for main struct - _tdrp_struct
     */

    typedef struct {

      size_t struct_size;

      /***** debug *****/

      tdrp_bool_t debug;

      /***** shape *****/

      _shape_t shape;

      /***** size *****/

      float size;

      /***** output_path *****/

      char* output_path;

    } _tdrp_struct;

    /*
     * function prototypes
     */

    extern int _tdrp_load_from_args(int argc, char **argv,
                                    _tdrp_struct *params,
                                    char **override_list,
                                    char **params_path_p);

    extern int _tdrp_load(char *param_file_path,
                          _tdrp_struct *params,
                          char **override_list,
                          int expand_env, int debug);

    extern int _tdrp_load_defaults(_tdrp_struct *params,
                                   int expand_env);

    extern void _tdrp_sync(void);

    extern void _tdrp_print(FILE *out, tdrp_print_mode_t mode);

    extern void _tdrp_free_all(void);

    extern int _tdrp_check_all_set(FILE *out);

    extern int _tdrp_array_realloc(char *param_name, int new_array_n);
     
    extern int _tdrp_array2D_realloc(char *param_name,
                                     int new_array_n1,
                                     int new_array_n2);

    extern TDRPtable *_tdrp_table(void);

    extern TDRPtable *_tdrp_init(_tdrp_struct *params);

    #ifdef __cplusplus
    }
    #endif

    #endif

------------------------------------------------------------------------

#### The TDRP C code - \_tdrp.c

    /*******************************************
     * _tdrp.c
     *
     * TDRP C code file.
     *
     * Code for program area_compute
     *
     * This file has been automatically
     * generated by TDRP, do not modify.
     *
     *******************************************/

    #include "_tdrp.h"
    #include 

    /*
     * file scope variables
     */

    static TDRPtable Table[5];
    static _tdrp_struct *Params;
    static char *Module = "";

    /*************************************************************
     * _tdrp_load_from_args()
     *
     * Loads up TDRP using the command line args.
     *
     * Check TDRP_usage() for command line actions associated with
     * this function.
     *
     *   argc, argv: command line args
     *
     *   _tdrp_struct *params: loads up this struct
     * 
     *   char **override_list: A null-terminated list of overrides
     *     to the parameter file.
     *     An override string has exactly the format of an entry
     *     in the parameter file itself.
     *
     *   char **params_path_p: if non-NULL, this is set to point to
     *                         the path of the params file used.
     *
     *  Returns 0 on success, -1 on failure.
     */

    int _tdrp_load_from_args(int argc, char **argv,
                             _tdrp_struct *params,
                             char **override_list,
                             char **params_path_p)
    {
      Params = params;
      _tdrp_init(Params);
      if (tdrpLoadFromArgs(argc, argv,
                           Table, Params,
                           override_list, params_path_p)) {
        return (-1);
      } else {
        return (0);
      }
    }

    /*************************************************************
     * _tdrp_load()
     *
     * Loads up TDRP for a given module.
     *
     * This version of load gives the programmer the option to load
     * up more than one module for a single application. It is a
     * lower-level routine than _tdrp_load_from_args,
     * and hence more flexible, but the programmer must do more work.
     *
     *   char *param_file_path: the parameter file to be read in.
     *
     *   _tdrp_struct *params: loads up this struct
     *
     *   char **override_list: A null-terminated list of overrides
     *     to the parameter file.
     *     An override string has exactly the format of an entry
     *     in the parameter file itself.
     *
     *  expand_env: flag to control environment variable
     *                expansion during tokenization.
     *              If TRUE, environment expansion is set on.
     *              If FALSE, environment expansion is set off.
     *
     *  Returns 0 on success, -1 on failure.
     */

    int _tdrp_load(char *param_file_path,
                   _tdrp_struct *params,
                   char **override_list,
                   int expand_env, int debug)
    {
      Params = params;
      _tdrp_init(Params);
      if (tdrpLoad(param_file_path, Table,
                   params, override_list,
                   expand_env, debug)) {
        return (-1);
      } else {
        return (0);
      }
    }

    /*************************************************************
     * _tdrp_load_defaults()
     *
     * Loads up defaults for a given module.
     *
     * See _tdrp_load() for more details.
     *
     *  Returns 0 on success, -1 on failure.
     */

    int _tdrp_load_defaults(_tdrp_struct *params,
                            int expand_env)
    {
      Params = params;
      _tdrp_init(Params);
      if (tdrpLoad(NULL, Table,
                   params, NULL,
                   expand_env, FALSE)) {
        return (-1);
      } else {
        return (0);
      }
    }

    /*************************************************************
     * _tdrp_sync()
     *
     * Syncs the user struct data back into the parameter table,
     * in preparation for printing.
     */

    void _tdrp_sync(void)
    {
      tdrpUser2Table(Table, Params);
    }

    /*************************************************************
     * _tdrp_print()
     * 
     * Print params file
     *
     * The modes supported are:
     *
     *   PRINT_SHORT:   main comments only, no help or descriptions
     *                  structs and arrays on a single line
     *   PRINT_NORM:    short + descriptions and help
     *   PRINT_LONG:    norm  + arrays and structs expanded
     *   PRINT_VERBOSE: long  + private params included
     */

    void _tdrp_print(FILE *out, tdrp_print_mode_t mode)
    {
      tdrpPrint(out, Table, Module, mode);
    }

    /*************************************************************
     * _tdrp_check_all_set()
     *
     * Return TRUE if all set, FALSE if not.
     *
     * If out is non-NULL, prints out warning messages for those
     * parameters which are not set.
     */

    int _tdrp_check_all_set(FILE *out)
    {
      return (tdrpCheckAllSet(out, Table, Params));
    }

    /*************************************************************
     * _tdrp_free_all()
     *
     * Frees up all TDRP dynamic memory.
     */

    void _tdrp_free_all(void)
    {
      tdrpFreeAll(Table, Params);
    }

    /*************************************************************
     * _tdrp_array_realloc()
     *
     * Realloc 1D array.
     *
     * If size is increased, the values from the last array entry is
     * copied into the new space.
     *
     * Returns 0 on success, -1 on error.
     */

    int _tdrp_array_realloc(char *param_name, int new_array_n)
    {
      if (tdrpArrayRealloc(Table, Params, param_name, new_array_n)) {
        return (-1);
      } else {
        return (0);
      }
    }

    /*************************************************************
     * _tdrp_array2D_realloc()
     *
     * Realloc 2D array.
     *
     * If size is increased, the values from the last array entry is
     * copied into the new space.
     *
     * Returns 0 on success, -1 on error.
     */

    int _tdrp_array2D_realloc(char *param_name,
                              int new_array_n1,
                              int new_array_n2)
    {
      if (tdrpArray2DRealloc(Table, Params, param_name,
                 new_array_n1, new_array_n2)) {
        return (-1);
      } else {
        return (0);
      }
    }

    /*************************************************************
     * _tdrp_table()
     *
     * Returns pointer to static Table for this module.
     */

    TDRPtable *_tdrp_table(void)
    {
      return (Table);
    }

    /*************************************************************
     * _tdrp_init()
     *
     * Module table initialization function.
     *
     *
     * Returns pointer to static Table for this module.
     */

    TDRPtable *_tdrp_init(_tdrp_struct *params)

    {

      TDRPtable *tt = Table;

      _tdrp_struct pp; /* for computing byte_offsets */

      /* zero out struct, and store size */

      memset(params, 0, sizeof(_tdrp_struct));
      params->struct_size = sizeof(_tdrp_struct);

      /* Parameter 'debug' */
      /* ctype is 'tdrp_bool_t' */
      
      memset(tt, 0, sizeof(TDRPtable));
      tt->ptype = BOOL_TYPE;
      tt->param_name = tdrpStrDup("debug");
      tt->descr = tdrpStrDup("Option to print debugging messages");
      tt->help = tdrpStrDup("");
      tt->val_offset = (char *) &(pp.debug) - (char *) &pp;
      tt->single_val.b = pFALSE;
      tt++;
      
      /* Parameter 'shape' */
      /* ctype is '_shape_t' */
      
      memset(tt, 0, sizeof(TDRPtable));
      tt->ptype = ENUM_TYPE;
      tt->param_name = tdrpStrDup("shape");
      tt->descr = tdrpStrDup("Shape type.");
      tt->help = tdrpStrDup("The program will compute the area of a square, circle or equilateral triangle.");
      tt->val_offset = (char *) &(pp.shape) - (char *) &pp;
      tt->enum_def.name = tdrpStrDup("shape_t");
      tt->enum_def.nfields = 3;
      tt->enum_def.fields = (enum_field_t *)
          tdrpMalloc(tt->enum_def.nfields * sizeof(enum_field_t));
        tt->enum_def.fields[0].name = tdrpStrDup("SQUARE");
        tt->enum_def.fields[0].val = SQUARE;
        tt->enum_def.fields[1].name = tdrpStrDup("CIRCLE");
        tt->enum_def.fields[1].val = CIRCLE;
        tt->enum_def.fields[2].name = tdrpStrDup("EQ_TRIANGLE");
        tt->enum_def.fields[2].val = EQ_TRIANGLE;
      tt->single_val.e = SQUARE;
      tt++;
      
      /* Parameter 'size' */
      /* ctype is 'float' */
      
      memset(tt, 0, sizeof(TDRPtable));
      tt->ptype = FLOAT_TYPE;
      tt->param_name = tdrpStrDup("size");
      tt->descr = tdrpStrDup("Size of the shape.");
      tt->help = tdrpStrDup("");
      tt->val_offset = (char *) &(pp.size) - (char *) &pp;
      tt->has_min = TRUE;
      tt->min_val.f = 0;
      tt->max_val.f = 1e+33;
      tt->single_val.f = 1;
      tt++;
      
      /* Parameter 'output_path' */
      /* ctype is 'string' */
      
      memset(tt, 0, sizeof(TDRPtable));
      tt->ptype = STRING_TYPE;
      tt->param_name = tdrpStrDup("output_path");
      tt->descr = tdrpStrDup("The path of the file to which the output is written.");
      tt->help = tdrpStrDup("The directory which contains this path must exist.");
      tt->val_offset = (char *) &(pp.output_path) - (char *) &pp;
      tt->single_val.s = tdrpStrDup("./area_compute.out");
      tt++;
      
      /* trailing entry has param_name set to NULL */
      
      tt->param_name = NULL;
      
      return (Table);

    }

<a id="section-example-cplusplus"></a>

## Example in C++

------------------------------------------------------------------------

#### The AreaCompute application

The program AreaCompute is a small application which demonstrates some of the TDRP functionality.

AreaCompute computes the area of a shape of a given size. The size is set in the parameter file. The user may choose one of three shapes, a square, circle or equilateral triangle. The area is computed and the result is written to a file. The file path is also set in the parameter file.

All of the parameters may be overridden by command line arguments.

------------------------------------------------------------------------

#### Paramdef file - paramdef.AreaCompute

    /*********************************************************
     * parameter definitions for AreaCompute
     *
     * Mike Dixon, RAP, NCAR, Boulder, CO, USA, 80307-3000
     *
     * Sept 1998
     */

    /*
     * AreaCompute is a small TDRP demonstration program.
     *
     * The program allows the user to compute the area of
     * a geometric shape of a given size.
     *
     * The result is printed to a file.
     */

    //////////////////////////////////////////////////////////

    paramdef boolean {
      p_default = FALSE;
      p_descr = "Option to print debugging messages";
    } debug;

    typedef enum {
      SQUARE, CIRCLE, EQ_TRIANGLE
    } shape_t ;

    paramdef enum shape_t {
      p_default = SQUARE;
      p_descr = "Shape type.";
      p_help = "The program will compute the area of a square,"
      "circle or equilateral triangle.";
    } shape;

    paramdef float {
      p_default = 1.0;
      p_min = 0.0;
      p_descr = "Size of the shape.";
    } size;

    paramdef string {
      p_default = "./AreaCompute.out";
      p_descr = "The path of the file to which the output is written.";
      p_help = "The directory which contains this path must exist.";
    } output_path;

------------------------------------------------------------------------

#### Parameter file - AreaCompute.params

The following was generated by running:  

      AreaCompute -print_params > file

    /**********************************************************************
     * TDRP params for AreaCompute
     **********************************************************************/

    ///////////// debug ///////////////////////////////////
    //
    // Option to print debugging messages.
    // Type: boolean
    //

    debug = FALSE;

    ///////////// shape ///////////////////////////////////
    //
    // Shape type.
    // The program will compute the area of a square, circle or equilateral 
    //   triangle.
    //
    // Type: enum
    // Options:
    //   SQUARE, CIRCLE, EQ_TRIANGLE
    //
    //

    shape = SQUARE;

    ///////////// size ////////////////////////////////////
    //
    // Size of the shape.
    // Minimum val: 0
    // Type: float
    //

    size = 1;

    ///////////// output_path /////////////////////////////
    //
    // The path of the file to which the output is written.
    // The directory which contains this path must exist.
    // Type: string
    //

    output_path = "./AreaCompute.out";

------------------------------------------------------------------------

#### Main driver - Main.cc

    #include "AreaCompute.hh"

    // file scope

    static void tidy_and_exit (int sig);
    static AreaCompute *Prog;

    // main

    int main(int argc, char **argv)

    {

      // create program object

      AreaCompute *Prog;
      Prog = new AreaCompute(argc, argv);
      if (!Prog->OK) {
        return(-1);
      }

      // run it

      int iret = Prog->Run();

      // clean up

      tidy_and_exit(iret);
      return (iret);
      
    }

    // tidy up on exit

    static void tidy_and_exit (int sig)

    {
      delete(Prog);
      exit(sig);
    }

------------------------------------------------------------------------

#### AreaCompute class header file - AreaCompute.hh

    #include 

    #include "Args.hh"
    #include "Params.hh"

    class AreaCompute {
      
    public:

      // constructor

      AreaCompute (int argc, char **argv);

      // destructor
      
      ~AreaCompute();

      // run 

      int Run();

      // data members

      int OK;

    protected:
      
    private:

      char *_progName;
      char *_paramsPath;
      Args *_args;
      Params *_params;
      double _area;

      int AreaCompute::_writeResults();

    };

------------------------------------------------------------------------

#### AreaCompute class code file - AreaCompute.cc

    #include "AreaCompute.hh"
    #include 
    #include 

    // Constructor

    AreaCompute::AreaCompute(int argc, char **argv)

    {

      OK = TRUE;

      // set programe name

      _progName = strdup("AreaCompute");

      // get command line args
      
      _args = new Args(argc, argv, _progName);
      if (!_args->OK) {
        fprintf(stderr, "ERROR: %s\n", _progName);
        fprintf(stderr, "Problem with command line args\n");
        OK = FALSE;
        return;
      }

      // get TDRP params

      _params = new Params();
      _paramsPath = "unknown";
      if (_params->loadFromArgs(argc, argv,
                    _args->override.list,
                    &_paramsPath)) {
        fprintf(stderr, "ERROR: %s\n", _progName);
        fprintf(stderr, "Problem with TDRP parameters\n");
        OK = FALSE;
        return;
      }

      return;

    }

    // destructor

    AreaCompute::~AreaCompute()

    {

      // free up

      delete(_params);
      delete(_args);
      free(_progName);
      
    }

    //////////////////////////////////////////////////
    // Run

    int AreaCompute::Run()
    {

      // compute area

      switch (_params->shape) {
        
      case SQUARE:
        _area = _params->size * _params->size;
        break;

      case CIRCLE:
        _area = _params->size * _params->size * (3.14159 / 4.0);
        break;

      case EQ_TRIANGLE:
        _area = _params->size * _params->size * (0.866 / 2.0);
        break;

      } // switch

      // debug message

      if (_params->debug) {
        fprintf(stderr, "Size is: %g\n", _params->size);
        switch (_params->shape) {
        case SQUARE:
          fprintf(stderr, "Shape is SQUARE\n");
          break;
        case CIRCLE:
          fprintf(stderr, "Shape is CIRCLE\n");
          break;
        case EQ_TRIANGLE:
          fprintf(stderr, "Shape is EQ_TRIANGLE\n");
          break;
        } /* switch */
        fprintf(stderr, "Area is: %g\n", _area);
      }
      
      // write out the result
      
      _writeResults();

      return (0);

    }

    //////////////////////////////////////////////////
    // writeResults

    int AreaCompute::_writeResults()

    {

      FILE *out;

      if ((out = fopen(_params->output_path, "w")) == NULL) {
        perror(_params->output_path);
        return (-1);
      }

      fprintf(out, "Size is: %g\n", _params->size);
      switch (_params->shape) {
      case SQUARE:
        fprintf(out, "Shape is SQUARE\n");
        break;
      case CIRCLE:
        fprintf(out, "Shape is CIRCLE\n");
        break;
      case EQ_TRIANGLE:
        fprintf(out, "Shape is EQ_TRIANGLE\n");
        break;
      } /* switch */
      fprintf(out, "Area is: %g\n", _area);

      fclose(out);

      return (0);

    }

------------------------------------------------------------------------

#### Parsing the command line - Args.hh

    #include 
    #include 

    class Args {
      
    public:

      // constructor

      Args (int argc, char **argv, char *prog_name);

      // Destructor

      ~Args();

      // public data

      int OK;
      tdrp_override_t override;

    protected:
      
    private:

      void _usage(char *prog_name, FILE *out);
      
    };

------------------------------------------------------------------------

#### Parsing the command line - Args.cc

    #include "Args.hh"
    #include 

    // Constructor

    Args::Args (int argc, char **argv, char *prog_name)

    {

      char tmp_str[BUFSIZ];

      // intialize

      OK = TRUE;
      TDRP_init_override(&override);
      
      // loop through args
      
      for (int i =  1; i < argc; i++) {

        if (!strcmp(argv[i], "-h")) {
      
          _usage(prog_name, stdout);
          exit(0);
          
        } else if (!strcmp(argv[i], "-debug")) {
          
          sprintf(tmp_str, "debug = TRUE;");
          TDRP_add_override(&override, tmp_str);
          
        } else if (!strcmp(argv[i], "-size")) {
          
          if (i < argc - 1) {
            sprintf(tmp_str, "size = %s;", argv[++i]);
        TDRP_add_override(&override, tmp_str);
          } else {
        OK = FALSE;
          }
        
        } else if (!strcmp(argv[i], "-shape")) {
          
          if (i < argc - 1) {
            sprintf(tmp_str, "shape = %s;", argv[++i]);
        TDRP_add_override(&override, tmp_str);
          } else {
        OK = FALSE;
          }
        
        } else if (!strcmp(argv[i], "-output")) {
          
          if (i < argc - 1) {
            sprintf(tmp_str, "output_path = %s;", argv[++i]);
        TDRP_add_override(&override, tmp_str);
          } else {
        OK = FALSE;
          }
        
        } // if
        
      } // i
      
      if (!OK) {
        _usage(prog_name, stderr);
      }
        
    }

    // Destructor

    Args::~Args ()

    {

      TDRP_free_override(&override);

    }
      
    void Args::_usage(char *prog_name, FILE *out)
    {

      fprintf(out, "%s%s%s%s",
          "Usage: ", prog_name, " [options as below]\n",
          "   [ -h] produce this list.\n"
          "   [ -debug ] print debug messages\n"
          "   [ -output ?] set output_path\n"
          "   [ -size ?] set shape size\n"
          "   [ -shape ?] set shape type\n"
          "      options are SQUARE, CIRCLE and EQ_TRIANGLE\n");

      TDRP_usage(out);

    }

------------------------------------------------------------------------

#### The Makefile

    PROGNAME = AreaCompute

    CPPC = g++
    RM = /bin/rm -f
    INCLUDES = -I../include
    CFLAGS = -g
    LDFLAGS = -L.. -ltdrp

    SRCS = \
        Params.cc \
        AreaCompute.cc \
        Args.cc \
        Main.cc

    OBJS = $(SRCS:.cc=.o)

    # link

    $(PROGNAME): $(OBJS)
        $(CPPC) -o $(PROGNAME) $(OBJS) $(LDFLAGS)

    # tdrp

    Params.cc: paramdef.$(PROGNAME)
        tdrp_gen -f paramdef.$(PROGNAME) -c++ -prog $(PROGNAME)

    clean_tdrp:
        $(RM) Params.hh Params.cc

    clean:
        $(RM) core a.out
        $(RM) *.i *.o  *.ln *~

    clean_bin:
        $(RM) $(PROGNAME)

    clean_all: clean clean_bin

    # suffix rules

    .SUFFIXES: .c .o

    .cc.o:
        $(CPPC) -c $(CFLAGS) $(INCLUDES) $<

------------------------------------------------------------------------

#### The TDRP Params class header file - Params.hh

    #include 
    #include 

    // typedefs

    // Class definition

    class Params {

    public:

      // enum typedefs

      typedef enum {
        SQUARE = 0,
        CIRCLE = 1,
        EQ_TRIANGLE = 2
      } shape_t;

      // Member functions

      Params() { _init(); }

      ~Params() { freeAll(); }

      int loadFromArgs(int argc, char **argv,
                       char **override_list,
                       char **params_path_p);

      int load(char *param_file_path,
               char **override_list,
               int expand_env, int debug);

      int loadDefaults(int expand_env);

      void sync();

      void print(FILE *out, tdrp_print_mode_t mode);

      int checkAllSet(FILE *out);

      int arrayRealloc(char *param_name,
                       int new_array_n);

      int array2DRealloc(char *param_name,
                         int new_array_n1,
                         int new_array_n2);

      void freeAll(void);

      // Data members

      char _start_; // start of data region
                    // needed for zeroing out data
                    // and computing offsets

      tdrp_bool_t debug;

      shape_t shape;

      float size;

      char* output_path;

      char _end_; // end of data region
                  // needed for zeroing out data

    private:

      void _init();

      TDRPtable _table[5];

      char *_className;
    };

------------------------------------------------------------------------

#### The TDRP Params class code - Params.cc

    #include "Params.hh"
    #include 

    //////////////////////////////////////////////////////////////
    // loadFromArgs()
    //
    // Loads up TDRP using the command line args.
    //
    // Check TDRP_usage() for command line actions associated with
    // this function.
    //
    //   argc, argv: command line args
    //
    //   char **override_list: A null-terminated list of overrides
    //     to the parameter file.
    //     An override string has exactly the format of an entry
    //     in the parameter file itself.
    //
    //   char **params_path_p: if non-NULL, this is set to point to
    //                         the path of the params file used.
    //
    //  Returns 0 on success, -1 on failure.
    //

    int Params::loadFromArgs(int argc, char **argv,
                             char **override_list,
                             char **params_path_p)
    {
      if (tdrpLoadFromArgs(argc, argv,
                           _table, &_start_,
                           override_list, params_path_p)) {
        return (-1);
      } else {
        return (0);
      }
    }

    //////////////////////////////////////////////////////////////
    // load()
    //
    // Loads up TDRP for a given class.
    //
    // This version of load gives the programmer the option to load
    // up more than one class for a single application. It is a
    // lower-level routine than loadFromArgs,
    // and hence more flexible, but the programmer must do more work.
    //
    //   char *param_file_path: the parameter file to be read in.
    //
    //   char **override_list: A null-terminated list of overrides
    //     to the parameter file.
    //     An override string has exactly the format of an entry
    //     in the parameter file itself.
    //
    //  expand_env: flag to control environment variable
    //                expansion during tokenization.
    //              If TRUE, environment expansion is set on.
    //              If FALSE, environment expansion is set off.
    //
    //  Returns 0 on success, -1 on failure.
    //

    int Params::load(char *param_file_path,
                     char **override_list,
                     int expand_env, int debug)
    {
      if (tdrpLoad(param_file_path,
                   _table, &_start_,
                   override_list,
                   expand_env, debug)) {
        return (-1);
      } else {
        return (0);
      }
    }

    //////////////////////////////////////////////////////////////
    // loadDefaults()
    //
    // Loads up default params for a given class.
    //
    // See load() for more detailed info.
    //
    //  Returns 0 on success, -1 on failure.
    //

    int Params::loadDefaults(int expand_env)
    {
      if (tdrpLoad(NULL,
                   _table, &_start_,
                   NULL, expand_env, FALSE)) {
        return (-1);
      } else {
        return (0);
      }
    }

    //////////////////////////////////////////////////////////////
    // sync()
    //
    // Syncs the user struct data back into the parameter table,
    // in preparation for printing.
    //

    void Params::sync(void)
    {
      tdrpUser2Table(_table, &_start_);
    }

    //////////////////////////////////////////////////////////////
    // print()
    // 
    // Print params file
    //
    // The modes supported are:
    //
    //   PRINT_SHORT:   main comments only, no help or descriptions
    //                  structs and arrays on a single line
    //   PRINT_NORM:    short + descriptions and help
    //   PRINT_LONG:    norm  + arrays and structs expanded
    //   PRINT_VERBOSE: long  + private params included
    //

    void Params::print(FILE *out, tdrp_print_mode_t mode)
    {
      tdrpPrint(out, _table, _className, mode);
    }

    //////////////////////////////////////////////////////////////
    // checkAllSet()
    //
    // Return TRUE if all set, FALSE if not.
    //
    // If out is non-NULL, prints out warning messages for those
    // parameters which are not set.
    //

    int Params::checkAllSet(FILE *out)
    {
      return (tdrpCheckAllSet(out, _table, &_start_));
    }

    //////////////////////////////////////////////////////////////
    // freeAll()
    //
    // Frees up all TDRP dynamic memory.
    //

    void Params::freeAll(void)
    {
      tdrpFreeAll(_table, &_start_);
    }

    //////////////////////////////////////////////////////////////
    // arrayRealloc()
    //
    // Realloc 1D array.
    //
    // If size is increased, the values from the last array entry is
    // copied into the new space.
    //
    // Returns 0 on success, -1 on error.
    //

    int Params::arrayRealloc(char *param_name, int new_array_n)
    {
      if (tdrpArrayRealloc(_table, &_start_,
                           param_name, new_array_n)) {
        return (-1);
      } else {
        return (0);
      }
    }

    //////////////////////////////////////////////////////////////
    // array2DRealloc()
    //
    // Realloc 2D array.
    //
    // If size is increased, the values from the last array entry is
    // copied into the new space.
    //
    // Returns 0 on success, -1 on error.
    //

    int Params::array2DRealloc(char *param_name,
                               int new_array_n1,
                               int new_array_n2)
    {
      if (tdrpArray2DRealloc(_table, &_start_, param_name,
                             new_array_n1, new_array_n2)) {
        return (-1);
      } else {
        return (0);
      }
    }

    //////////////////////////////////////////////////////////////
    // _init()
    //
    // Class table initialization function.
    //
    //

    void Params::_init()

    {

      TDRPtable *tt = _table;

      // zero out table

      memset(_table, 0, sizeof(_table));

      // zero out members

      memset(&_start_, 0, &_end_ - &_start_);

      _className = "Params";
      // Parameter 'debug'
      // ctype is 'tdrp_bool_t'
      
      memset(tt, 0, sizeof(TDRPtable));
      tt->ptype = BOOL_TYPE;
      tt->param_name = tdrpStrDup("debug");
      tt->descr = tdrpStrDup("Option to print debugging messages");
      tt->help = tdrpStrDup("");
      tt->val_offset = (char *) &debug - &_start_;
      tt->single_val.b = pFALSE;
      tt++;
      
      // Parameter 'shape'
      // ctype is '_shape_t'
      
      memset(tt, 0, sizeof(TDRPtable));
      tt->ptype = ENUM_TYPE;
      tt->param_name = tdrpStrDup("shape");
      tt->descr = tdrpStrDup("Shape type.");
      tt->help = tdrpStrDup("The program will compute the area of a square, circle or equilateral triangle.");
      tt->val_offset = (char *) &shape - &_start_;
      tt->enum_def.name = tdrpStrDup("shape_t");
      tt->enum_def.nfields = 3;
      tt->enum_def.fields = (enum_field_t *)
          tdrpMalloc(tt->enum_def.nfields * sizeof(enum_field_t));
        tt->enum_def.fields[0].name = tdrpStrDup("SQUARE");
        tt->enum_def.fields[0].val = SQUARE;
        tt->enum_def.fields[1].name = tdrpStrDup("CIRCLE");
        tt->enum_def.fields[1].val = CIRCLE;
        tt->enum_def.fields[2].name = tdrpStrDup("EQ_TRIANGLE");
        tt->enum_def.fields[2].val = EQ_TRIANGLE;
      tt->single_val.e = SQUARE;
      tt++;
      
      // Parameter 'size'
      // ctype is 'float'
      
      memset(tt, 0, sizeof(TDRPtable));
      tt->ptype = FLOAT_TYPE;
      tt->param_name = tdrpStrDup("size");
      tt->descr = tdrpStrDup("Size of the shape.");
      tt->help = tdrpStrDup("");
      tt->val_offset = (char *) &size - &_start_;
      tt->has_min = TRUE;
      tt->min_val.f = 0;
      tt->max_val.f = 1e+33;
      tt->single_val.f = 1;
      tt++;
      
      // Parameter 'output_path'
      // ctype is 'string'
      
      memset(tt, 0, sizeof(TDRPtable));
      tt->ptype = STRING_TYPE;
      tt->param_name = tdrpStrDup("output_path");
      tt->descr = tdrpStrDup("The path of the file to which the output is written.");
      tt->help = tdrpStrDup("The directory which contains this path must exist.");
      tt->val_offset = (char *) &output_path - &_start_;
      tt->single_val.s = tdrpStrDup("./AreaCompute.out");
      tt++;
      
      // trailing entry has param_name set to NULL
      
      tt->param_name = NULL;
      
      return;

    }

<a id="section-makefiles"></a>

## TDRP Makefiles

#### Makefile for single TDRP C module

Assume that you are building the program tdrp_test, which has a couple of subroutines, do_printout() and parse_args(), each of which is in a separate C file. The main is in tdrp_test.c. You need to generate the files \_tdrp.h and \_tdrp.c, which will contain the TDRP code.

The following fragment from the Makefile shows how to do this:

    ###########################################################################
    #
    # Makefile fragment for tdrp_test program
    #
    ###########################################################################

    LDFLAGS = -ltdrp

    OBJS = \
        _tdrp.o \
        do_printout.o \
        parse_args.o \
        tdrp_test.o

    _tdrp.c: paramdef.tdrp_test
        tdrp_gen -f paramdef.tdrp_test -prog tdrp_test

    clean_tdrp:
        $(RM) _tdrp.h _tdrp.c

Note that \_tdrp.o is at the top of the OBJS list. This forces the generation of \_tdrp.h *before* compiling the other files, since the program code will need the header file.

#### Makefile for multiple TDRP C modules per program

Assume that you are building the program tdrp_mult, which has two TDRP modules. You need to generate the files mod1_tdrp.h and mod1_tdrp.c for the module mod1, and mod2_tdrp.h and mod2_tdrp.c for the module mod2.

The following fragment from the Makefile shows how to do this:

    ###########################################################################
    #
    # Makefile fragment for tdrp_mult program
    #
    ###########################################################################

    LDFLAGS = -ltdrp

    OBJS = \
        mod1_tdrp.o \
        mod2_tdrp.o \
        do_printout.o \
        parse_args.o \
        tdrp_mult.o

    mod1_tdrp.c: paramdef.mod1
        tdrp_gen mod1 -f paramdef.mod1 -prog tdrp_test

    mod2_tdrp.c: paramdef.mod2
        tdrp_gen mod2 -f paramdef.mod2 -prog tdrp_test

    clean_tdrp:
        $(RM) mod1_tdrp.h mod1_tdrp.c mod2_tdrp.h mod2_tdrp.c

#### Makefile for single TDRP C++ class

Assume that you are building the program TdrpTest, which has a couple of classes Args and TdrpTest, and a Main routine. You need to generate the files Params.hh and Params.cc, which will contain the TDRP class.

The following fragment from the Makefile shows how to do this:

    ###########################################################################
    #
    # Makefile fragment for TdrpTest program
    #
    ###########################################################################

    LDFLAGS = -ltdrp

    OBJS = \
        Params.o \
        Args.o \
        Main.o \
        TdrpTest.o

    Params.cc: paramdef.TdrpTest
        tdrp_gen -f paramdef.TdrpTest -c++ -prog TdrpTest

    clean_tdrp:
        $(RM) Params.hh Params.cc

Note that Params.o is at the top of the OBJS list. This forces the generation of Params.hh *before* compiling the other files, since the program code will need the header file.

#### Makefile for multiple TDRP C++ classes

Assume that you are building the program TdrpMult, which needs 2 TDRP classes, Class1 and Class2. You need to generate the files Class1.hh, Class1.cc, Class2.hh and Class2.cc. The following fragment from the Makefile shows how to do this:

    ###########################################################################
    #
    # Makefile fragment for TdrpMult program
    #
    ###########################################################################

    LDFLAGS = -ltdrp

    OBJS = \
        Class1.o \
        Class2.o \
        Args.o \
        Main.o \
        TdrpTest.o

    Class1.cc: paramdef.Class1
        tdrp_gen -f paramdef.Class1 -c++ -class Class1 -prog TdrpTest

    Class2.cc: paramdef.Class2
        tdrp_gen -f paramdef.Class2 -c++ -class Class2 -prog TdrpTest

    clean_tdrp:
        $(RM) Class1.hh Class1.cc Class2.hh Class2.cc

<a id="section-advantages"></a>

## Advantages of using TDRP

- Self-documenting. All parameter documentation is done in the paramdef file, therefore the documentation and code are co-located, making it easy to keep the documentation up-to-date.
- Automatic generation of a default parameter file. Use of the `-print_params` feature allows the new user to generate a default parameter file. The documentation included in the file helps the new user to understand the use of the program and customize the parameters. Users find this one of the most useful aspects of the system.
- Checking of parameters which have not been set by the user. Use of the `-check_params` feature allows the user to check which parameters have not been explicitly set.
- Simple default behavior. If no parameter file is used, the default parameter values are used.
- Support for environment variables in the parameter files. Use of environment variables along with parameters provides a powerful combination for maintaining simplicity while porting applications from one environment to another.
- Automatic range checking of numeric variables. If the `p_min` and `p_max` features are set, the range of the user's parameters are checked to ensure they lie within the specified range.
- Allows a simple transition from development to deployment. The use of the `p_private` feature allows the programmer to lock parameters at their default values without changing any code. The programmer has the advantages of parameter settings during development while being assured that they will not be set to invalid values after deployment.
- Simple and robust. Completely implemented in C therfore no fancy compilers or libraries are required.
- Explicit support for both C and C++. Support for most native C variable types.
- Minimal coding required. Most work is done in the the paramdef file.

<a id="section-history"></a>

## TDRP history

#### 1993

John Yunker created the RTP (Run-Time Parameters) system, consisting of the rtp library and a program for generating code. rtp was based on lex and yacc and created a number of table files which were needed by the running executable.

#### 1994

John Caron reworked RTP into TDRP (Table-Driven Runtime Parameters). TDRP was still based on lex and yacc. tdrp_gen generated 3 header files and 1 C-code file. The run-time tables were no longer required.

#### 1995 - 1997

Mike Dixon ported TDRP to Dec Alpha and Linux, fixing some memory bugs along the way. The number of generated files was reduced from 4 to 2, one header and one C code. The Slackware Linux port was straightfoward. However, later Linux versions used bison for yacc, and tdrp_gen exhibited a bug which we have never been able to solve. Therefore the Slackware yacc and lex programs had to be installed for TDRP to work.

#### 1998

Mike Dixon rewrote TDRP in vanilla C to remove the dependency on yacc and lex which was causing problems on Linux. The system was enhanced in a number of ways, and C++ class code generation was added.

<a id="section-parameter-file-example"></a>

## Parameter File Example

### Example TDRP parameter file

    /**********************************************************************
     * TDRP params for tdrp_example
     **********************************************************************/

    //======================================================================
    //
    // INTEGER PARAMETERS.
    //
    // Testing integer parameter behavior.
    //
    //======================================================================
     
    ///////////// your_age ////////////////////////////////
    //
    // Single int value.
    // Testing single int actions.
    // Minimum val: 0
    // Maximum val: 120
    // Type: int
    //

    your_age = 35;

    ///////////// our_ages ////////////////////////////////
    //
    // Int array - variable length.
    // Testing variable length int array.
    // Minimum val: 0
    // Maximum val: 120
    // Type: int
    // 1D array - variable length - 5 elements.
    //

    our_ages = { 30, 31, 42, 43, 54 };

    ///////////// icon ////////////////////////////////////
    //
    // Variable length 2-D array.
    // Testing variable length 2-D array.
    // Minimum val: 0
    // Maximum val: 1
    // Type: int
    // 2D array - variable size (4 x 5).
    //

    icon = {
      { 0, 0, 1, 1, 1 },
      { 0, 0, 0, 0, 1 },
      { 0, 1, 0, 1, 0 },
      { 0, 0, 0, 1, 1 }
    };

    //======================================================================
    //
    // LONG INTEGER PARAMETERS.
    //
    // Testing long integer parameter behavior.
    //
    //======================================================================
     
    ///////////// number_of_radars ////////////////////////
    //
    // Single long value.
    // Testing single long actions.
    // Minimum val: 0
    // Type: long
    //

    number_of_radars = 1;

    ///////////// days_in_month ///////////////////////////
    //
    // Long array - fixed length.
    // Testing fixed length long array.
    // Type: long
    // 1D array - fixed length - 12 elements.
    //

    days_in_month = { 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };

    ///////////// item_count //////////////////////////////
    //
    // Variable fixed 2-D array.
    // Testing fixed length 2-D array.
    // Minimum val: 0
    // Type: long
    // 2D array - fixed size (4 x 6).
    //

    item_count = {
      { 0, 5, 6, 11, 2, 3 },
      { 9, 8, 15, 12, 4, 4 },
      { 17, 18, 3, 7, 0, 12 },
      { 15, 10, 10, 1, 9, 1 }
    };

    //======================================================================
    //
    // FLOAT PARAMETERS.
    //
    // Testing float parameter behavior.
    //
    //======================================================================
     
    ///////////// speed ///////////////////////////////////
    //
    // Single float value.
    // Testing single float actions.
    // Minimum val: 0
    // Type: float
    //

    speed = 15;

    ///////////// storm_volume ////////////////////////////
    //
    // Float array - fixed length.
    // Testing fixed length float array.
    // Type: float
    // 1D array - fixed length - 10 elements.
    //

    storm_volume = { 101.1, 102.1, 103.1, 104.1, 105.1, 106.1, 107.1, 108.1, 109.1, 110.1 };

    ///////////// rain_accumulation ///////////////////////
    //
    // Variable length 2-D array.
    // Testing variable length 2-D array.
    // Type: float
    // 2D array - variable size (4 x 5).
    //

    rain_accumulation = {
      { 0.1, 0.6, 1.9, 12.4, 1.1 },
      { 2.3, 5.7, 12.8, 19.4, 0 },
      { 14.3, 19.3, 12.1, 3.3, 7.5 },
      { 8, 6.1, 0, 15.1, 10 }
    };

    //======================================================================
    //
    // DOUBLE PARAMETERS.
    //
    // Testing double parameter behavior.
    //
    //======================================================================
     
    ///////////// mass_coefficient ////////////////////////
    //
    // Single double value.
    // Testing single double actions.
    // Type: double
    //

    mass_coefficient = 9.1e-09;

    ///////////// storm_mass //////////////////////////////
    //
    // Double array - variable length.
    // Testing variable length double array.
    // Minimum val: 1
    // Type: double
    // 1D array - variable length - 5 elements.
    //

    storm_mass = { 1.9e+08, 2.1e+08, 9.7e+07, 5.3e+07, 1.1e+09 };

    ///////////// length_factor ///////////////////////////
    //
    // Fixed length 2-D array.
    // Testing fixed length 2-D array.
    // Type: double
    // 2D array - fixed size (3 x 6).
    //

    length_factor = {
      { 0.9, 0.9, 1.9, 1.9, 1.9, 100.3 },
      { 0.9, 1.9, 0.9, 1.9, 0.9, -100.1 },
      { 0.9, 0.9, 0.9, 1.9, 1.9, -99.9 }
    };

    //======================================================================
    //
    // BOOLEAN PARAMETERS.
    //
    // Testing boolean parameter behavior.
    //
    //======================================================================
     
    ///////////// use_data ////////////////////////////////
    //
    // Single bool value.
    // Testing single bool actions.
    // Type: boolean
    //

    use_data = TRUE;

    ///////////// allow_outliers //////////////////////////
    //
    // Bool array - variable length.
    // Testing variable length bool array.
    // Type: boolean
    // 1D array - variable length - 4 elements.
    //

    allow_outliers = { TRUE, FALSE, TRUE, TRUE };

    ///////////// compute_length //////////////////////////
    //
    // Variable length 2-D array.
    // Testing variable length 2-D array.
    // Type: boolean
    // 2D array - variable size (4 x 5).
    //

    compute_length = {
      { FALSE, FALSE, TRUE, TRUE, TRUE },
      { FALSE, FALSE, FALSE, FALSE, TRUE },
      { FALSE, TRUE, FALSE, TRUE, FALSE },
      { FALSE, FALSE, FALSE, TRUE, TRUE }
    };

    ///////////// debug ///////////////////////////////////
    //
    // Option to print debugging messages.
    // Type: boolean
    //

    debug = FALSE;

    ///////////// flags ///////////////////////////////////
    //
    // Test boolean flags.
    // Type: boolean
    // 1D array - fixed length - 6 elements.
    //

    flags = { TRUE, FALSE, TRUE, FALSE, TRUE, TRUE };

    //======================================================================
    //
    // STRING PARAMETERS.
    //
    // Testing string parameter behavior.
    //
    //======================================================================
     
    ///////////// input_file_ext //////////////////////////
    //
    // Input file extension.
    // Testing single-valued string parameter.
    // Type: string
    //

    input_file_ext = "mcg";

    ///////////// input_file_paths ////////////////////////
    //
    // Input file paths.
    // Testing variable length array of strings. Note imbedded environment 
    //   variables.
    // Type: string
    // 1D array - variable length - 3 elements.
    //

    input_file_paths = { "$(HOME)/path1", "$(HOME)/paths", "$(HOME)/path3" };

    ///////////// output_file_paths ///////////////////////
    //
    // Output file paths.
    // Testing variable length 2D array of strings. Note imbedded environment 
    //   variables.
    // Type: string
    // 2D array - variable size (6 x 3).
    //

    output_file_paths = {
      { "$(USER)/path11", "$(USER)/path21", "$(USER)/path31" },
      { "$(USER)/path12", "$(USER)/path22", "$(USER)/path32" },
      { "$(USER)/path13", "$(USER)/path23", "$(USER)/path33" },
      { "$(USER)/path14", "$(USER)/path24", "$(USER)/path34" },
      { "$(USER)/path15", "$(USER)/path25", "$(USER)/path35" },
      { "$(USER)/path16", "$(USER)/path26", "$(USER)/path36" }
    };

    ///////////// input_dir ///////////////////////////////
    //
    // Input directory.
    // Path of input directory - realtime mode onlyNote imbedded environment 
    //   variables.
    // Type: string
    //

    input_dir = "$(HOME)/input_dir";

    //======================================================================
    //
    // ENUM PARAMETERS.
    //
    // Testing enum parameter behavior.
    //
    //======================================================================
     
    ///////////// data_origin /////////////////////////////
    //
    // Data origin position.
    // Testing variable length enum array.
    //
    // Type: enum
    // Options:
    //   BOTLEFT, TOPLEFT, BOTRIGHT, TOPRIGHT
    //
    // 1D array - variable length - 2 elements.
    //

    data_origin = { BOTLEFT, TOPLEFT };

    ///////////// mode ////////////////////////////////////
    //
    // Testing 2-D enum array.
    // The options for this enum are defined in the paramdef instead of in a 
    //   typedef.
    //
    // Type: enum
    // Options:
    //   REALTIME, ARCHIVE, OTHER
    //
    // 2D array - fixed size (2 x 4).
    //

    mode = {
      { REALTIME, REALTIME, ARCHIVE, OTHER },
      { OTHER, ARCHIVE, ARCHIVE, REALTIME }
    };

    //======================================================================
    //
    // STRUCT PARAMETERS.
    //
    // Testing struct parameter behavior.
    //
    //======================================================================
     
    ///////////// grid ////////////////////////////////////
    //
    // Grid parameters.
    // Testing single-valued struct.Struct Definition occurs within the 
    //   paramdef.
    //
    // Type: struct
    //   typedef struct {
    //      long nx;
    //      long ny;
    //      double minx;
    //      double miny;
    //      double dx;
    //      double dy;
    //   }
    //
    //

    grid = { 100, 100, -50, -50, 2, 2.5 };

    ///////////// surface_stations ////////////////////////
    //
    // Surface station information.
    // Test of variable length struct array. Note that the struct is defined 
    //   in a typedef before the paramdef. Also, the struct includes an enum 
    //   which is pre-defined. Enums include din this manned MUST be defined in 
    //   a typedef.
    //
    // Type: struct
    //   typedef struct {
    //      double lat;
    //      double lon;
    //      double wind_sensor_ht;
    //      gauge_t gauge_make;
    //      boolean has_humidity;
    //   }
    //
    // 1D array - fixed length - 3 elements.
    //

    surface_stations = {
      { 40.1012, -104.231, 10, ETI, TRUE},
      { 40.2109, -104.576, 10, GEONOR, FALSE},
      { 39.1379, -104.908, 3, CAMPBELL, FALSE}
    };

    ///////////// data_field //////////////////////////////
    //
    // Data field parameters.
    // Test of fixed-length struct array.
    //
    // Type: struct
    //   typedef struct {
    //      double scale;
    //      double bias;
    //      long nplanes;
    //      string name;
    //      string units;
    //      origin_t origin;
    //   }
    //
    // 1D array - variable length - 2 elements.
    //

    data_field = {
      { 0.5, 1, 16, "Reflectivity", "dBZ", BOTLEFT},
      { 0.6, 1.1, 17, "Velocity", "m/s", TOPLEFT}
    };

<a id="section-api-c-old"></a>

## Old TDRP API for C

This API is included for backward compatibility. Use of this old API is discouraged because it will eventually be discontinued.

    /*************************************************************************
     * TDRP_load_from_args()
     *
     * Loads up TDRP using the command line args for control.
     *
     * Check TDRP_usage() for command line actions associated with
     * this function.
     *
     *   argc, argv: command line args
     *
     *   TDRPtable *table: table obtained from _tdrp_init().
     *
     *   void *params: this is actually of type *_tdrp_struct,
     *     as declared in _tdrp.h.
     *     This function loads the values of the parameters into this structure.
     * 
     *   char **override_list: A null-terminated list of overrides to the
     *     parameter file.
     *     An override string has exactly the format of the
     *     parameter file itself.
     *
     *   char **params_path_p: if non-NULL, this is set to point to the
     *                         path of the params file used.
     *
     *  Returns 0 on success, -1 on failure.
     */

    extern int TDRP_load_from_args(int argc, char **argv,
                       TDRPtable *table, void *params,
                       char **override_list,
                       char **params_path_p);

    /*************************************************************************
     * TDRP_load()
     *
     * Loads up TDRP for a given module.
     *
     * This version of load gives the programmer the option to load up more
     * than one module for a single application. It is a lower-level
     * routine than tdrpLoadFromArgs(), and hence more flexible, but
     * the programmer must do more work.
     *
     *   char *param_file_path: the parameter file to be read in.
     *
     *   TDRPtable *table: table obtained from _tdrp_init().
     *
     *   void *params: this is actually of type *_tdrp_struct,
     *     as declared in _tdrp.h.
     *     This function loads the values of the parameters into this structure.
     * 
     *   char **override_list: A null-terminated list of overrides to the
     *     parameter file.
     *     An override string has exactly the format of the
     *     parameter file itself.
     *
     *  expand_env: flag to control environment variable expansion during
     *                tokenization.
     *              If TRUE, environment expansion is set on.
     *              If FALSE, environment expansion is set off.
     *
     *  Returns 0 on success, -1 on failure.
     */

    extern int TDRP_load(char *param_file_path,
                 TDRPtable *table, void *params,
                 char **override_list,
                 int expand_env, int debug);

    /**********************************************************
     * TDRP_load_defaults()
     *
     * Loads up TDRP for a given module using defaults only.
     *
     * See TDRP_load() for details.
     *
     * Returns 0 on success, -1 on failure.
     */

    extern int TDRP_load_defaults(TDRPtable *table, void *params,
                      int expand_env);

    /***********************************************************
     * TDRP_sync()
     *
     * Syncs the user struct data back into the parameter table,
     * in preparation for printing.
     */

    extern void TDRP_sync(TDRPtable *table, void *params);

    /*************
     * TDRP_print()
     * 
     * Print params file
     *
     * The modes supported are:
     *
     *   PRINT_SHORT:   main comments only, no help or descriptions
     *                  structs and arrays on a single line
     *   PRINT_NORM:    short + descriptions and help
     *   PRINT_LONG:    norm  + arrays and structs expanded
     *   PRINT_VERBOSE: long  + private params included
     */

    extern void TDRP_print(FILE *out, TDRPtable *table,
                   char *module, tdrp_print_mode_t mode);
         
    /***********************************************************
     * TDRP_check_all_set()
     *
     * Return TRUE if all set, FALSE if not.
     *
     * If out is non-NULL, prints out warning messages for those
     * parameters which are not set.
     */

    extern int TDRP_check_all_set(FILE *out,
                      TDRPtable *table, void *params);

    /***********************************************************
     * TDRP_free_all()
     *
     * Frees up memory associated with a module and its table
     */

    extern void TDRP_free_all(TDRPtable *table, void *params);

    /**********************
     * TDRP_array_realloc()
     *
     * Realloc 1D array.
     *
     * If size is increased, the values from the last array entry is
     * copied into the new space.
     *
     * Returns 0 on success, -1 on error.
     */

    extern int TDRP_array_realloc(TDRPtable *table, void *params,
                      char *param_name, int new_array_n);

    /************************
     * TDRP_array2D_realloc()
     *
     * Realloc 2D array.
     *
     * If size is increased, the values from the last array entry is
     * copied into the new space.
     *
     * Returns 0 on success, -1 on error.
     */

    extern int TDRP_array2D_realloc(TDRPtable *table, void *params,
                    char *param_name,
                    int new_array_n1, int new_array_n2);

    /***********************************************************
     * TDRP_str_replace()
     *
     * Replace a string.
     */

    extern void TDRP_str_replace(char **s1, char *s2);

    /**********************
     * TDRP_init_override()
     *
     * Initialize the override list
     */

    extern void TDRP_init_override(tdrp_override_t *override);

    /*********************
     * TDRP_add_override()
     *
     * Add a string to the override list
     */

    extern void TDRP_add_override(tdrp_override_t *override, char *override_str);

    /**********************
     * TDRP_free_override()
     *
     * Free up the override list
     */

    extern void TDRP_free_override(tdrp_override_t *override);

    /*********************************************************
     * TDRP_usage()
     *
     * Prints out usage message for TDRP args as passed in to
     * TDRP_load_from_args()
     */

    extern void TDRP_usage(FILE *out);
