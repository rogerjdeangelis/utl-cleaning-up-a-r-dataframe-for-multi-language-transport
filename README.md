# utl-cleaning-up-a-r-dataframe-for-multi-language-transport
Cleaning up a r dataframe for multi language transport
    %let pgm=utl-cleaning-up-a-r-dataframe-for-multi-language-transport;

    Cleaning up a r dataframe for multi language transport

    github
    https://tinyurl.com/2s35m5an
    https://github.com/rogerjdeangelis/utl-cleaning-up-a-r-dataframe-for-multi-language-transport

    The biggest issue I have with R is creating clean transport files.
    There are so many R only data types and exotic data structures that are
    not supported in other languages.

    Below is method to convert a R dataframe with NULLs, NaN, NA, Inf -Inf and Factors
    data types to something SAS can use. Impossible for R data sructures.
    Python is worse.

         Method

           1. Replace  NULLs, NaN, NA, Inf, and -Inf with NA
           2. Replace Factors with Character type
           3. Clean up other data issues by washing the dataframe through SQLite.
              Exporting to a in memory SQlite database file and then importing cleaned up.
              Peeserves table and datatypes that are compatible.


    Related
    Python data frame to R dataframe,
    https://tinyurl.com/2rn9fyt7
    https://github.com/rogerjdeangelis/utl-exporting-python-panda-dataframes-to-wps-r-using-a-shared-sqllite-database

    /*                   _
    (_)_ __  _ __  _   _| |_
    | | `_ \| `_ \| | | | __|
    | | | | | |_) | |_| | |_
    |_|_| |_| .__/ \__,_|\__|
            |_|
    */

    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*  R DATAFRAME (LESS WOULD BE MORE)                                                                                      */
    /*                                                                                                                        */
    /*   A      B      C    D      E      F     G                                                                             */
    /*  <fct> <fct> <dbl> <dbl>  <chr>  <lgl> <dbl>                                                                           */
    /*  ===== ===== ===== =====  =====  ===== =====                                                                           */
    /*                                                                                                                        */
    /*  1     <NA>    NA  -Inf     U     NaN    NaN                                                                           */
    /*  2     <NA>     1   Inf     A     NULL   NULL                                                                          */
    /*  3     P        2    NA     <NA>  NULL   NULL                                                                          */
    /*  4     V        3     3     C     NULL   NULL                                                                          */
    /*  5     NaN      4     4     Z     NA     NULL                                                                          */
    /*                                                                                                                        */
    /* CONTENTS                                                                                                               */
    /*                                                                                                                        */
    /* tibble [5 x 7] (S3: tbl_df/tbl/data.frame)                                                                             */
    /*                                                                                                                        */
    /*  $ a: Factor w/ 5 levels "1","2","3","4",..: 1 2 3 4                                                                   */
    /*  $ b: Factor w/ 3 levels "NaN","P","V": NA NA 2 3 1                                                                    */
    /*  $ c: num [1:5] NA 1 2 3 4                                                                                             */
    /*  $ d: num [1:5] -Inf Inf NA 3 4                                                                                        */
    /*  $ e: chr [1:5] "U" "A" NA "C" ...                                                                                     */
    /*  $ f: logi [1:5] NA NA NA NA NA        ** Converted from NULL to NA by dplyr?                                          */
    /*  $ g: num [1:5] NaN NaN NaN NaN NaN    ** Converted from NULL to NaN by dplyr?                                         */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    /*           _               _
      ___  _   _| |_ _ __  _   _| |_
     / _ \| | | | __| `_ \| | | | __|
    | (_) | |_| | |_| |_) | |_| | |_
     \___/ \__,_|\__| .__/ \__,_|\__|
                    |_|
    */

    /**************************************************************************************************************************/
    /*                              |                                                                                         */
    /*                              |                                                                                         */
    /*  CLEANED UP R DATAFRAME      |  Up to 40 obs from SD1.DF_XPORT total obs=5 27MAY2023:16:40:25                          */
    /*                              |                                                                                         */
    /*                              |  Obs    A    B    C    D    E    F    G                                                 */
    /*    a    b  c  d    e  f  g   |                                                                                         */
    /*  1 1 <NA> NA NA    U NA NA   |   1     1         .    .    U    .    .                                                 */
    /*  2 2 <NA>  1 NA    A NA NA   |   2     2         1    .    A    .    .                                                 */
    /*  3 3    P  2 NA <NA> NA NA   |   3     3    P    2    .         .    .                                                 */
    /*  4 4    V  3  3    C NA NA   |   4     4    V    3    3    C    .    .                                                 */
    /*  5 5 <NA>  4  4    Z NA NA   |   5     5         4    4    Z    .    .                                                 */
    /*                              |                                                                                         */
    /*                              |   #    Variable    Type    Len                                                          */
    /*                              |                                                                                         */
    /*                              |   1    A           Char      1                                                          */
    /*                              |   2    B           Char      2                                                          */
    /*                              |   3    C           Num       8                                                          */
    /*                              |   4    D           Num       8                                                          */
    /*                              |   5    E           Char      2                                                          */
    /*                              |   6    F           Num       8                                                          */
    /*                              |   7    G           Num       8                                                          */
    /*                              |                                                                                         */
    /**************************************************************************************************************************/

    /*
     _ __  _ __ ___   ___ ___  ___ ___
    | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | |_) | | | (_) | (_|  __/\__ \__ \
    | .__/|_|  \___/ \___\___||___/___/
    |_|
    */

    %utl_submit_wps64('
    libname sd1 "d:/sd1";
    proc r;
     submit;
     library(RSQLite);
     library(DBI);
     library(dplyr);
     con <- dbConnect(RSQLite::SQLite(), ":memory:");
      df <- dplyr::tibble(
         a = as.factor(c(1, 2, 3, 4, 5))
        ,b = as.factor(c(NA, NA, "P", "V", NA))
        ,c = c(NA,1,2,3,4)
        ,d = c(-Inf,Inf,NA,3,4)
        ,e = c("U","A", NA, "C", "Z")
        ,f = c(NULL,NULL,NULL,NULL,NA)
        ,g = c(NaN,NULL,NULL,NULL,NULL)
        );
     str(df);
     df[sapply(df, is.infinite)] <- NA;
     is.nan.data.frame <- function(x) do.call(cbind, lapply(x, is.nan));
     str(df);
     df[is.nan(df)] <- NA;
     dbWriteTable(con, "df", df);
     df_xport<-dbReadTable(con, "df");
     str(df_xport);
     df_xport;
     endsubmit;
     import data=sd1.df_xport r=df_xport;
     run;quit;
    ');

     proc print data=sd1.df_xport;
     run;quit;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
