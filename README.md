REPORT ZRPR_PRACTICE02.

********************************************************************************************************************
*                                   Tables Declaration
********************************************************************************************************************
TABLES : MARC,MARA,MARD.

TYPE-POOLS: SLIS.
********************************************************************************************************************
*                                   Internal tables Declaration
********************************************************************************************************************

TYPES: BEGIN OF TY_FINAL,
          MATNR TYPE MARC-MATNR,
          WERKS TYPE MARC-WERKS,
          PSTAT TYPE MARC-PSTAT,
          EKGRP TYPE MARC-EKGRP,
          ERSDA TYPE MARA-ERSDA,
          ERNAM TYPE MARA-ERNAM,
          LAEDA TYPE MARA-LAEDA,
          MATKL TYPE MARA-MATKL,
          LABST TYPE MARD-LABST,
          LFGJA TYPE MARD-LFGJA,
       END OF TY_FINAL.

DATA : IT_FINAL TYPE STANDARD TABLE OF TY_FINAL,
       WA_FINAL TYPE TY_FINAL.

********************************************************************************************************************
*                                   Selection  Screen
********************************************************************************************************************

START-OF-SELECTION.

  SELECTION-SCREEN BEGIN OF BLOCK B1 WITH FRAME TITLE TEXT-001.
  SELECT-OPTIONS: S_MATNR FOR MARC-MATNR.
  SELECT-OPTIONS: S_WERKS FOR MARC-WERKS.
  SELECTION-SCREEN END OF BLOCK B1.

END-OF-SELECTION.


********************************************************************************************************************
*                                   Start of selection  Screen
********************************************************************************************************************

START-OF-SELECTION.

  SELECT MATNR,
         WERKS,
         PSTAT,
         EKGRP
     FROM MARC
     INTO TABLE @DATA(IT_MARC)
     WHERE MATNR IN @S_MATNR
     AND WERKS IN @S_WERKS.

  IF IT_MARC IS NOT INITIAL.
    SELECT MATNR,
           ERSDA,
           ERNAM,
           LAEDA,
           MATKL
      FROM MARA
      INTO TABLE @DATA(IT_MARA)
      FOR ALL ENTRIES IN @IT_MARC
      WHERE MATNR = @IT_MARC-MATNR.
  ENDIF.

  IF IT_MARA IS NOT INITIAL.
    SELECT MATNR,
           WERKS,
           LFGJA,
           LABST
     FROM MARD
     INTO TABLE @DATA(IT_MARD)
     FOR ALL ENTRIES IN @IT_MARA
     WHERE MATNR = @IT_MARA-MATNR
*     AND WERKS = @IT_MARC-WERKS
     AND LGORT EQ '1000'.
  ENDIF.

END-OF-SELECTION.
********************************************************************************************************************
*                                             LOOP
********************************************************************************************************************
LOOP AT IT_MARA INTO DATA(WA_MARA).

  WA_FINAL-ERSDA = WA_MARA-ERSDA.
  WA_FINAL-ERNAM = WA_MARA-ERNAM.
  WA_FINAL-LAEDA = WA_MARA-LAEDA.
  WA_FINAL-MATKL = WA_MARA-MATKL.

 READ TABLE IT_MARC INTO DATA(WA_MARC) WITH KEY MATNR = WA_MARA-MATNR.


 IF SY-SUBRC = 0.

  WA_FINAL-MATNR = WA_MARC-MATNR.
  WA_FINAL-WERKS = WA_MARC-WERKS.
  WA_FINAL-PSTAT = WA_MARC-PSTAT.
  WA_FINAL-EKGRP = WA_MARC-EKGRP.

 ENDIF.

  READ TABLE IT_MARD INTO DATA(WA_MARD) WITH KEY MATNR = WA_MARA-MATNR.

   APPEND WA_FINAL TO IT_FINAL.
    CLEAR : WA_FINAL.

ENDLOOP.



**********************************************************************************************************************
*                                        Field Catlogue
**********************************************************************************************************************

*DATA: IT_FIELDCAT TYPE SLIS_T_FIELDCAT_ALV WITH HEADER LINE.

 DATA: IT_FIELDCAT TYPE SLIS_T_FIELDCAT_ALV WITH HEADER LINE,
        WA_FIELDCAT TYPE SLIS_T_FIELDCAT_ALV WITH HEADER LINE.
  DATA: T_LAYOUT       TYPE SLIS_LAYOUT_ALV.

  IT_FIELDCAT-FIELDNAME   = 'MATNR'.
  IT_FIELDCAT-SELTEXT_M   = 'Material Number'.
  IT_FIELDCAT-COL_POS     = 1.
  APPEND IT_FIELDCAT.
  CLEAR  IT_FIELDCAT.

  IT_FIELDCAT-FIELDNAME   = 'WERKS'.
  IT_FIELDCAT-SELTEXT_M   = 'Plant'.
  IT_FIELDCAT-COL_POS     = 2.
  APPEND IT_FIELDCAT.
  CLEAR IT_FIELDCAT.

  IT_FIELDCAT-FIELDNAME   = 'PSTAT'.
  IT_FIELDCAT-SELTEXT_M   = 'Maintenance status'.
  IT_FIELDCAT-COL_POS     = 3.
  APPEND IT_FIELDCAT.
  CLEAR IT_FIELDCAT.

  IT_FIELDCAT-FIELDNAME   = 'EKGRP'.
  IT_FIELDCAT-SELTEXT_M   = 'Purchasing Group'.
  IT_FIELDCAT-COL_POS     = 4.
  APPEND IT_FIELDCAT.
  CLEAR IT_FIELDCAT.

  IT_FIELDCAT-FIELDNAME   = 'ERSDA'.
  IT_FIELDCAT-SELTEXT_M   = 'Created On'.
  IT_FIELDCAT-COL_POS     = 5.
  APPEND IT_FIELDCAT.
  CLEAR IT_FIELDCAT.

  IT_FIELDCAT-FIELDNAME   = 'ERNAM'.
  IT_FIELDCAT-SELTEXT_M   = 'Name of Person who Created the Object'.
  IT_FIELDCAT-COL_POS     = 6.
  APPEND IT_FIELDCAT.
  CLEAR IT_FIELDCAT.

  IT_FIELDCAT-FIELDNAME   = 'LAEDA'.
  IT_FIELDCAT-SELTEXT_M   = 'Date of Last Change'.
  IT_FIELDCAT-COL_POS     = 7.
  APPEND IT_FIELDCAT.
  CLEAR IT_FIELDCAT.

  IT_FIELDCAT-FIELDNAME   = 'MATKL'.
  IT_FIELDCAT-SELTEXT_M   = 'Material Group'.
  IT_FIELDCAT-COL_POS     = 8.
  APPEND IT_FIELDCAT.
  CLEAR IT_FIELDCAT.

  IT_FIELDCAT-FIELDNAME   = 'LABST'.
  IT_FIELDCAT-SELTEXT_M   = 'Valuated Unrestricted-Use Stock'.
  IT_FIELDCAT-COL_POS     = 9.
  APPEND IT_FIELDCAT.
  CLEAR IT_FIELDCAT.

  IT_FIELDCAT-FIELDNAME   = 'LFGJA'.
  IT_FIELDCAT-SELTEXT_M   = 'Fiscal Year of Current Period'.
  IT_FIELDCAT-COL_POS     = 10.
  APPEND IT_FIELDCAT.
  CLEAR IT_FIELDCAT.


***********************************************************************************************************************
*                                             Function Module
***********************************************************************************************************************
  CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
    EXPORTING
      I_CALLBACK_PROGRAM = SY-REPID
*      IS_LAYOUT          = T_LAYOUT
      IT_FIELDCAT        = IT_FIELDCAT[]
      I_DEFAULT          = 'X'
      I_SAVE             = 'A'
    TABLES
      T_OUTTAB           = IT_FINAL
    EXCEPTIONS
      PROGRAM_ERROR      = 1
      OTHERS             = 2.

  IF SY-SUBRC <> 0.
  ENDIF.
