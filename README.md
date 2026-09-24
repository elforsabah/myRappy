    "-----------------------------------------------------------------------
    " Path prefix of the BMS API (e.g. /BmsApiSapTest) for a destination,
    " from ZTOUR_BMS_CFG. Empty when not maintained. No trailing slash.
    "-----------------------------------------------------------------------
    CLASS-METHODS get_path_prefix
      IMPORTING iv_destination   TYPE rfcdest
      RETURNING VALUE(rv_prefix) TYPE string.


  METHOD get_path_prefix.

    CLEAR rv_prefix.

    SELECT SINGLE bms_path_prefix
      FROM ztour_bms_cfg
      WHERE bms_destination = @iv_destination
      INTO @DATA(lv_prefix).

    IF sy-subrc <> 0.
      RETURN.
    ENDIF.

    rv_prefix = condense( CONV string( lv_prefix ) ).

    IF rv_prefix IS NOT INITIAL
       AND substring( val = rv_prefix off = strlen( rv_prefix ) - 1 ) = '/'.
      rv_prefix = substring( val = rv_prefix len = strlen( rv_prefix ) - 1 ).
    ENDIF.

  ENDMETHOD.
