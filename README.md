# CBT093
Converted to GitHub via [cbt2git](https://github.com/wizardofzos/cbt2git)

This is still a work in progress. 
Due to amazing work by Alison Zhang and Jake Choi repos are no longer deleted.

```
//***FILE 093 Contains a system to sequentialize a PDS (any         *   FILE 093
//*           LRECL) from Sam Golob.  Some of this code is from     *   FILE 093
//*           the SHARE PL/I tape.  For additional information,     *   FILE 093
//*           see the members called $$$$DOC and $$$$DOC2.          *   FILE 093
//*                                                                 *   FILE 093
//*           Gerhard Postpischil has submitted a new version       *   FILE 093
//*           of PDSLOAD, current member PDSLOADW.  All older       *   FILE 093
//*           versions of PDSLOAD have been renumbered in order     *   FILE 093
//*           as follows:  PDSLOOLD, the oldest version, followed   *   FILE 093
//*           by PDSLOAD1 thru PDSLOAD9 in date order.  PDSLOADW    *   FILE 093
//*           is newer than all of them.  I guess that I'll         *   FILE 093
//*           adhere to this scheme.  (SBG - 2012/02/20)            *   FILE 093
//*                                                                 *   FILE 093
//*           THIS SYSTEM SHOULD PROVE USEFUL IF YOU WANT           *   FILE 093
//*           TO "SEQUENTIALIZE" A PDS.                             *   FILE 093
//*                                                                 *   FILE 093
//*           THIS LIBRARY CONTAINS TWO PAIRS OF PROGRAMS:          *   FILE 093
//*                                                                 *   FILE 093
//*           1.  OFFLOADW AND PDSLOADW    (MODIFIED FROM           *   FILE 093
//*               CBT TAPE BY GREG PRICE, et al)                    *   FILE 093
//*                                                                 *   FILE 093
//*           2.  LISTPDS can be used instead of OFFLOADW,          *   FILE 093
//*               and LISTPDS has just been fixed to accommodate    *   FILE 093
//*               extended ISPF statistics.  (Version 8.2)          *   FILE 093
//*               (June 2016)                                       *   FILE 093
//*                                                                 *   FILE 093
//*           See members $$NOTEnn for more history about           *   FILE 093
//*               this file.                                        *   FILE 093
//*                                                                 *   FILE 093
//*     07/2022   Fixed OFFLOAD, LISTPDS for ISPF statistics        *   FILE 093
//*               which contain packed dates ending in X'0C'.       *   FILE 093
//*               (v10.2)                                           *   FILE 093
//*                                                                 *   FILE 093
//*     02/2017   Fixed OFFLOAD, LISTPDS for 8-character            *   FILE 093
//*               ISPF userids in the ISPF stats.  (v10.1, v8.3)    *   FILE 093
//*                                                                 *   FILE 093
//*     10/2015   Fixed V10.0 - OFFLOADW optionally puts out a      *   FILE 093
//*     V10.0     //SYSUPLOG DD name, which logs all records        *   FILE 093
//*               in pds members that originally have string "><"   *   FILE 093
//*               in column 1, so that they don't get changed       *   FILE 093
//*               back to string "./" by PDSLOADW.                  *   FILE 093
//*                                                                 *   FILE 093
//*     09/2013   These two programs, PDSLOADW and OFFLOADW,        *   FILE 093
//*               have now been fixed to handle extended            *   FILE 093
//*               ISPF stats if they exist (from z/OS 1.11 on).     *   FILE 093
//*                                                                 *   FILE 093
//*     02/2017   Fixed PDSLOADW, OFFLOADW, and LISTPDS to handle   *   FILE 093
//*               pds member ISPF userids of 8 characters.          *   FILE 093
//*                                                                 *   FILE 093
//*     11/2018   LISTPDS was fixed to punch ./ ALIAS cards if      *   FILE 093
//*               it is run with a PARM of 'ALIAS'.                 *   FILE 093
//*                                                                 *   FILE 093
//*           Extended format of the ./ ADD card:                   *   FILE 093
//*                                                                 *   FILE 093
//*    From columns 01 thru 68 is the old format of the ./ ADD card *   FILE 093
//*                                                                 *   FILE 093
//*    Column 72 has to be a space, to accommodate IEBUPDTE ./      *   FILE 093
//*     continuations, if they exist.                               *   FILE 093
//*                                                                 *   FILE 093
//*    From columns 70 thru 80 is the new format, as follows:       *   FILE 093
//*                                                                 *   FILE 093
//*    ----+----1----+----2-                                        *   FILE 093
//*    ./ ADD NAME=membname                                         *   FILE 093
//*                                                                 *   FILE 093
//*    ---+----3----+----4----+----5----+----6----+----7----+----8  *   FILE 093
//*    vvmm-crdat-moddt-hhmm-sssss-iiiii-mmmmm-MYUSRID ss nnniiimm  *   FILE 093
//*    ver  yyjjj yyjjj time size  init  modif userid               *   FILE 093
//*    mod                                                          *   FILE 093
//*                          last                                   *   FILE 093
//*                          five   same  same                      *   FILE 093
//*                          digits                                 *   FILE 093
//*                                                                 *   FILE 093
//*           where ss  is decimal digits 1-2 of seconds of time    *   FILE 093
//*           where nnn is decimal digits 6-8 of the ISPF size      *   FILE 093
//*                   (so maximum number is 99,999,999).            *   FILE 093
//*           where iii is decimal digits 6-8 of the ISPF init      *   FILE 093
//*                   (so maximum number is 99,999,999).            *   FILE 093
//*           where mm  is decimal digits 6-7 of the ISPF modified  *   FILE 093
//*                   (so maximum number is 9,999,999).             *   FILE 093
//*           userid can now be up to 8 characters, filling in      *   FILE 093
//*                   the space at character 69.                    *   FILE 093
//*                                                                 *   FILE 093
//*           2.  UNUPDTE AND UPDTE       (FROM PL1 MODS            *   FILE 093
//*               TAPE - SPLA.  UNUPDTE WAS ENHANCED BY ART         *   FILE 093
//*               TANSKY OF SUNGARD.)                               *   FILE 093
//*                                                                 *   FILE 093
//*           Note: Small fix to PDSLOADX from Gerd Petermann:      *   FILE 093
//*                 email:  GPetermann@horizont-it.com              *   FILE 093
//*                                                                 *   FILE 093
//*           EACH PAIR IS A SELF-CONTAINED SYSTEM THAT IS          *   FILE 093
//*           INDEPENDENT OF THE OTHER PAIR.                        *   FILE 093
//*                                                                 *   FILE 093
//*           OFFLOADW AND PDSLOADW ALLOW IEBUPDTE-TYPE UNLOADING   *   FILE 093
//*           AND RELOADING OF PDS'ES TO SEQUENTIAL DATASETS.       *   FILE 093
//*           THIS IS NOT RESTRICTED TO RECORD LENGTHS OF 80 FOR    *   FILE 093
//*           THE DATA.  ALMOST ANY PARTITIONED DATASETS ARE        *   FILE 093
//*           ELIGIBLE FOR THIS TREATMENT.  THIS OPENS              *   FILE 093
//*           IEBUPDTE-TYPE UNLOADS TO TAPE OR DISK-SEQUENTIAL      *   FILE 093
//*           DATASETS TO MUCH WIDER APPLICATION THAN HERETOFORE.   *   FILE 093
//*           (ALSO SEE THE "=OFFLOAD" OPTION OF THE "REVIEW" TSO   *   FILE 093
//*           COMMAND THAT IS ON FILE 134 OF THIS TAPE.)            *   FILE 093
//*                                                                 *   FILE 093
//*           OFFLOAD AND PDSLOAD NOW AUTOMATICALLY ALLOW           *   FILE 093
//*           FOR LRECL FROM 1 TO 256 NOW (FROM GREG PRICE)         *   FILE 093
//*           WITH NO CONDITIONAL ASSEMBLY.                         *   FILE 093
//*                                                                 *   FILE 093
//*           THE RESULT WAS ACHIEVED BY SLIGHTLY MODIFYING         *   FILE 093
//*           EXISTING PROGRAMS PDSLOAD (FROM CBT TAPE FILE         *   FILE 093
//*           316) AND OFFLOAD (FROM CBT TAPE FILE 225).            *   FILE 093
//*           YOU CAN GET THE IEBUPDTE-TYPE UNLOAD AND              *   FILE 093
//*           RELOAD TREATMENT, COMPLETE WITH THE   ./ ADD          *   FILE 093
//*           CARDS AND ISPF STATISTICS PRESERVED.  AFTER           *   FILE 093
//*           GREG PRICE'S MODIFICATIONS, OFFLOAD AND PDSLOAD       *   FILE 093
//*           ARE NOW A MATCHED PAIR OF PROGRAMS TO PERFORM         *   FILE 093
//*           OPPOSITE FUNCTIONS:  OFFLOAD SEQUENTIALIZES A PDS,    *   FILE 093
//*           AND PDSLOAD RELOADS THE PDS FROM THE SEQUENTIAL       *   FILE 093
//*           OFFLOADED FILE.                                       *   FILE 093
//*                                                                 *   FILE 093
//*           THE UPDTE AND UNUPDTE PROGRAMS WERE LIFTED            *   FILE 093
//*           FROM THE PL1 MODS TAPE THAT CAN BE OBTAINED           *   FILE 093
//*           FROM SPLA (ORDER NUMBER 370D-03.2.019).               *   FILE 093
//*                                                                 *   FILE 093
//*           THESE PROGRAMS ARE MORE FLEXIBLE THAN                 *   FILE 093
//*           PDSLOADW AND OFFLOADW IN THAT:                        *   FILE 093
//*                                                                 *   FILE 093
//*            1.  THEY HANDLE RECFM=F AND ALSO RECFM=V             *   FILE 093
//*                DATASETS.                                        *   FILE 093
//*                                                                 *   FILE 093
//*            2.  THE DATASET CAN HAVE ANY LRECL PERMITTED         *   FILE 093
//*                BY THE SYSTEM.                                   *   FILE 093
//*                                                                 *   FILE 093
//*        PROCESSING WITH THESE PROGRAMS SEQUENTIALIZES A          *   FILE 093
//*        PDS BY LOADING EACH MEMBER TO A SEQUENTIAL               *   FILE 093
//*        DATASET, PRECEDED BY A CONTROL RECORD THAT LOOKS         *   FILE 093
//*        LIKE   ./ ADD NAME=MEMBNAME , SIMILAR TO AN              *   FILE 093
//*        IEBUPDTE CONTROL CARD.                                   *   FILE 093
//*                                                                 *   FILE 093
//*        THE PROGRAM UNUPDTE CONVERTS A PARTITIONED               *   FILE 093
//*        DATASET INTO SEQUENTIAL FORMAT DESCRIBED BY THE          *   FILE 093
//*        PRECEDING PARAGRAPH.  THE PROGRAM UPDTE LOADS            *   FILE 093
//*        THE SEQUENTIALIZED DATASET OF THE ABOVE FORMAT           *   FILE 093
//*        BACK INTO A PDS THAT HAS THE SAME DCB ATTRIBUTES         *   FILE 093
//*        (EXCEPT FOR DSORG OF COURSE).                            *   FILE 093
//*                                                                 *   FILE 093
//*        IT IS ALSO ADVANTAGEOUS TO HAVE PDSLOADW AND             *   FILE 093
//*        OFFLOADW AROUND, BECAUSE THEY HAVE SOME OPTIONS          *   FILE 093
//*        WHICH UPDTE AND UNUPDTE DO NOT HAVE, SUCH AS             *   FILE 093
//*        AUTOMATICALLY CONVERTING THE STRING ./ WITHIN A          *   FILE 093
//*        MEMBER (IN COLUMNS 1-2) TO SOME OTHER STRING,            *   FILE 093
//*        SUCH AS ><.  THEREFORE I AM INCLUDING BOTH PAIRS         *   FILE 093
//*        OF PROGRAMS IN THIS PACKAGE.                             *   FILE 093
//*                                                                 *   FILE 093
//*        JCL TO RUN THESE PROGRAMS IS OF THE SAME FORMAT          *   FILE 093
//*        AS IEBUPDTE JCL, TO THE POINT WHERE EACH PROGRAM         *   FILE 093
//*        MIMICS THE FUNCTION OF IEBUPDTE.  FOR INSTANCE,          *   FILE 093
//*        UNUPDTE, WHICH UNLOADS A PDS TO A SEQUENTIAL             *   FILE 093
//*        DATASET, HAS DDCARDS SYSPRINT, SYSUT1, AND               *   FILE 093
//*        SYSUT2.  UPDTE, WHICH DOES THE OPPOSITE, HAS             *   FILE 093
//*        CONTROL CARDS SYSPRINT, SYSIN, AND SYSUT2.  YOU          *   FILE 093
//*        GET THE PICTURE.                                         *   FILE 093
//*                                                                 *   FILE 093
//*        THESE FOUR PROGRAMS (TWO PAIRS) TAKEN TOGETHER,          *   FILE 093
//*        PROVIDE POWERFUL TOOLS FOR SEQUENTIALIZATION OF          *   FILE 093
//*        PARTITIONED DATASETS.                                    *   FILE 093
//*                                                                 *   FILE 093
//*  -------------------------------------------------------------  *   FILE 093
//*                                                                 *   FILE 093
//*  PDSLOAD update notes from Greg Price:  (see File 134 - REVIEW) *   FILE 093
//*                                                                 *   FILE 093
//*       PDSLOAD has now been further enhanced to handle any       *   FILE 093
//*       LRECL for both fixed-length and variable-length           *   FILE 093
//*       records.  The LRECL of the input sequential data set      *   FILE 093
//*       can but need not match the LRECL of the output            *   FILE 093
//*       partitioned data set.                                     *   FILE 093
//*                                                                 *   FILE 093
//*       When the output PDS has fixed-length records, the input   *   FILE 093
//*       file may have fixed-length or variable-length records.    *   FILE 093
//*       (Text files transferred from PCs often go to variable-    *   FILE 093
//*       length record files on MVS.)                              *   FILE 093
//*                                                                 *   FILE 093
//*       When the output PDS has variable-length records, only     *   FILE 093
//*       variable-length record input data is acceptable.          *   FILE 093
//*                                                                 *   FILE 093
//*       Undefined record format files cannot be used for input    *   FILE 093
//*       or output.                                                *   FILE 093
//*                                                                 *   FILE 093
//*       The minimum input LRECL is 80.  The minimum output        *   FILE 093
//*       LRECL is 1 (plus 4 for RDWs, if present).                 *   FILE 093
//*                                                                 *   FILE 093
//*       PARM=NEW is used to specify that, like IEBUPDTE, the      *   FILE 093
//*       input control+data stream is to be loaded from SYSIN,     *   FILE 093
//*       instead of SYSUT1.  In any event, if an OPEN for SYSUT1   *   FILE 093
//*       does not open successfully (and no abend occurs) the      *   FILE 093
//*       OPEN is retried with SYSIN as the DDname.                 *   FILE 093
//*                                                                 *   FILE 093
//*       PARM=SPF can still be used to generate ISPF               *   FILE 093
//*       "statistics".  SSI information will be lost when this     *   FILE 093
//*       is selected.  The "userid" of generated stats is          *   FILE 093
//*       'PDSLOAD'.                                                *   FILE 093
//*                                                                 *   FILE 093
//*       John Kalinich's Y2K windowing fix allows for 2-digit      *   FILE 093
//*       years below 66 to be deemed to belong to the 21st         *   FILE 093
//*       century.  This is necessary because the PDSLOAD stats     *   FILE 093
//*       format on the ./ ADD statement only allows for 2-digit    *   FILE 093
//*       years.  (Generated stats did not have a Y2K bug.)         *   FILE 093
//*                                                                 *   FILE 093
//*       The SPF stats current record count will always be set     *   FILE 093
//*       from the record count processed by PDSLOAD, even when     *   FILE 093
//*       this differs from the data supplied on a ./ ADD card.     *   FILE 093
//*       Other data will not be overridden.  Apart from the        *   FILE 093
//*       userid, supplied stats are now verified to consist of     *   FILE 093
//*       numeric characters.                                       *   FILE 093
//*                                                                 *   FILE 093
//*       The asterisk (*), question mark (?) and percent sign      *   FILE 093
//*       (%) are now treated as generic character placeholders     *   FILE 093
//*       for member selection.  The three mask characters          *   FILE 093
//*       function identically, and cause a match for the           *   FILE 093
//*       corresponding byte position of the member name.  Thus,    *   FILE 093
//*       S(ABC****X) will select all members beginning with        *   FILE 093
//*       'ABC' and ending in 'X' in the eighth byte, and S(****)   *   FILE 093
//*       will select all members with names no longer than four    *   FILE 093
//*       non-blank characters.                                     *   FILE 093
//*                                                                 *   FILE 093
//*       The IBM OS utility DDname override parameter can now be   *   FILE 093
//*       used by PDSLOAD.  The SYSIN, SYSPRINT and SYSUT2          *   FILE 093
//*       "slots" are relevant.  (This was done to facilitate       *   FILE 093
//*       dynamic invocation from the REVIEW TSO command.)  See     *   FILE 093
//*       the leading comments in the source code for more          *   FILE 093
//*       information on DDname overrides.                          *   FILE 093
//*                                                                 *   FILE 093
//*       Parameter order is 'NEW,SPF,S(********),UPDTE(><)' for    *   FILE 093
//*       example.  Unwanted options can be omitted, but the        *   FILE 093
//*       order is fixed.                                           *   FILE 093
//*                                                                 *   FILE 093
//*    Greg Price    20 April, 1999                                 *   FILE 093
//*                                                                 *   FILE 093
//*    Note from Sam Golob    03 April, 2012                        *   FILE 093
//*                                                                 *   FILE 093
//*     Greg Price and Gerhard Postpischil have the same initials.  *   FILE 093
//*     Therefore, Greg refers to this "update team" as TEAM-GP.    *   FILE 093
//*                                                                 *   FILE 093
//*     One issue dealt with in PDSLOAD is that it checks           *   FILE 093
//*     member names for "validity" before it loads a new pds       *   FILE 093
//*     member with that name.  I took a vote among two people:     *   FILE 093
//*     Greg Price said to eliminate the validity check for         *   FILE 093
//*     member names altogether.  Gerhard Postpischil said to       *   FILE 093
//*     keep it in.  Gerhard was willing to put his money where     *   FILE 093
//*     his mouth is, so he programmed a new version of PDSLOAD,    *   FILE 093
//*     now presented as member PDSLOADW.                           *   FILE 093
//*                                                                 *   FILE 093
//*     Gerhard's new version has options.  Default is to           *   FILE 093
//*     eliminate the validity check.  Two other options are to     *   FILE 093
//*     either check the name in a limited manner, or check the     *   FILE 093
//*     name in a stricter manner.                                  *   FILE 093
//*                                                                 *   FILE 093
//*        PARM='NAME=ASIS'   BYPASS ALL CHECKS  (default)          *   FILE 093
//*             'NAME=CHECK'  ALLOW ALL PRINTABLE CHARACTERS        *   FILE 093
//*                           (EXCEPT COMMA) USING CODEPAGE 037     *   FILE 093
//*             'NAME=IBM'    ENFORCE STRICT IBM JCL STANDARDS      *   FILE 093
//*                                                                 *   FILE 093
```
