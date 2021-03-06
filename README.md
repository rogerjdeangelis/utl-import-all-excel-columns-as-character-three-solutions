# utl-import-all-excel-columns-as-character-three-solutions
Import all excel columns as character three solutions
    Import all excel columns as character three solutions

     Three Solutions

        1. SAS  libname engine
        2. R  xlsx package  (SAS/IML/R)
        3. SAS Passthru


    github (this solution)
    https://tinyurl.com/ybx6lpp9
    https://github.com/rogerjdeangelis/utl-import-all-excel-columns-as-character-three-solutions

    github
    https://tinyurl.com/y8fb3kqe
    https://communities.sas.com/t5/SAS-Data-Management/Reading-XLSX-file-and-change-the-all-column-type-into-character/m-p/523523

    Related repos

    github
    https://github.com/rogerjdeangelis/utl-excel-fixing-bad-formatting-using-passthru

    SAS  Forum
    https://communities.sas.com/t5/SAS-Programming/PROC-IMPORT-XLS-engine-mixed-columns/m-p/502853

    github
    https://tinyurl.com/ydb98fjx
    https://github.com/rogerjdeangelis/utl_excel_import_all_columns_as_character_and_preserve_long_variable_names

    see SAS Forum
    https://tinyurl.com/y6twrqlu
    https://communities.sas.com/t5/SAS-Programming/Further-process-after-PROC-IMPORT/m-p/497835

    Other excel repositories
    https://tinyurl.com/ybnm6azh
    https://github.com/rogerjdeangelis?utf8=%E2%9C%93&tab=repositories&q=excel+in%3Aname&type=&language=



    INPUT
    =====

    MAKE DATA

      %utlfkil(d:\xls\class.xlsx);
      libname xl 'd:\xls\class.xlsx';
      data xl.class;
        set sashelp.class;
      run;quit;
      libname xl clear;


      d:/xls/class.xlsx


         +----------------------------------------------------------------+
         |     A      |    B       |     C      |    D       |    E       |
         +----------------------------------------------------------------+
       1 | NAME       |   SEX      |    AGE     |  HEHT    |  WEHT        |
         +------------+------------+------------+------------+------------+
       2 | ALFRED     |    M       |    14      |    69      |  112.5     |
         +------------+------------+------------+------------+------------+
          ...
         +------------+------------+------------+------------+------------+
       N | WILLIAM    |    M       |    15      |   66.5     |  112       |
         +------------+------------+------------+------------+------------+

       [class]


    EXAMPLE OUTPUT
    --------------

    SAS  libname
    ------------

    WORK.SASCLASS total obs=19

           Variables in Creation Order

      #    Variable    Type    Len    Format

      1    NAME        Char      8    $8.
      2    SEX         Char      1    $1.
      3    AGE         Char     10    $10.
      4    HEIGHT      Char     10    $10.
      5    WEIGHT      Char     10    $10.


    Obs    NAME       SEX    AGE    HEIGHT        WEIGHT

      1    Alfred      M     14     69            112.5
      2    Alice       F     13     56.5          84
      3    Barbara     F     13     65.2999999    98
      4    Carol       F     14     62.7999999    102.5
      5    Henry       M     14     63.5          102.5
      6    James       M     12     57.2999999    83


    R  xlsx package
    ---------------

     Variables in Creation Order

    #    Variable    Type    Len

    1    NAME        Char      7
    2    SEX         Char      1
    3    AGE         Char      2
    4    HEIGHT      Char      4
    5    WEIGHT      Char      5


    WORK.RCLASS total obs=19

    Obs   NAME       SEX    AGE    HEIGHT    WEIGHT

     1    Alfred      M     14      69       112.5
     2    Alice       F     13      56.5     84
     3    Barbara     F     13      65.3     98
     4    Carol       F     14      62.8     102.5
     5    Henry       M     14      63.5     102.5
     6    James       M     12      57.3     83
     7    Jane        F     12      59.8     84.5
    ...


    SAS Passthru
    ------------

          Variables in Creation Order

    #    Variable    Type     Len    Format

    1    NAME        Char     255    $255.
    2    SEX         Char     255    $255.
    3    AGE         Char    1024    $1024.
    4    HEIGHT      Char    1024    $1024.
    5    WEIGHT      Char    1024    $1024.

    After macro utl_optlen

    #    Variable    Type    Len

    1    NAME        Char      7
    2    SEX         Char      1
    3    AGE         Char      2
    4    HEIGHT      Char      4
    5    WEIGHT      Char      5

    WORK.PASSAS total obs=19

     Obs    NAME       SEX    AGE    HEIGHT    WEIGHT

       1    Alfred      M     14      69.0     112.5
       2    Alice       F     13      56.5     84.0
       3    Barbara     F     13      65.3     98.0
       4    Carol       F     14      62.8     102.5
       5    Henry       M     14      63.5     102.5
       6    James       M     12      57.3     83.0
      ....

    PROCESS
    =======


     1. SAS  libname engine
     -----------------------

        libname xl  'd:\xls\class.xlsx' scan_text=no ;
        data work.sasClass;
        set xl.class(
                dbsastype=(
                    name='char(8)'
                    sex='char(1)'
                    age='char(10)'
                    height='char(10)'
                    weight='char(10)'
         ));
        run;
        libname xl  clear;


     2. R  xlsx package  (SAS/IML/R)
     -------------------------------

        %utl_submit_r64('
           library(xlsx);
           library(Hmisc);
           library(SASxport);
           class<-read.xlsx("d:/xls/class.xlsx",1,colClasses=rep("character",16),stringsAsFactors=FALSE);
           class;
           write.xport(class,file="d:/xpt/rxpt.xpt");
        ');

         libname xpt xport "d:/xpt/rxpt.xpt";
              data rclass;
                set xpt.class;
              run;quit;
         libname xpt clear;


    3. SAS Passthru
    ---------------

        proc sql dquote=ansi;
         connect to excel
            (Path="d:/xls/class.xlsx" );
            create
                table pasSas as
            select
                *
                from connection to Excel
                (
                 Select
                    name
                   ,sex
                   ,format(age,'##') as age
                   ,format(height,'###.0') as height
                   ,format(weight,'###.0') as weight
                 from
                   [class]
                );
            disconnect from Excel;
        Quit;

        * fix the 255 and 1024 lengths;
        %utl_optlen(inp=pasSas,out=pasSas);

    OUTPUT
    see above


