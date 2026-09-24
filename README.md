  METHOD get_path_prefix.

    CLEAR rv_prefix.

    " Same row the action reads: CONFIG_ID = DEFAULT. The destination is
    " only checked when the row names one, so a wrong destination in the
    " call cannot silently pick up the wrong prefix.
    SELECT SINGLE bms_destination, bms_path_prefix
      FROM ztour_bms_cfg
      WHERE config_id = 'DEFAULT'
      INTO @DATA(ls_cfg).

    IF sy-subrc <> 0.
      RETURN.
    ENDIF.

    IF ls_cfg-bms_destination IS NOT INITIAL
       AND ls_cfg-bms_destination <> iv_destination.
      RETURN.
    ENDIF.

    rv_prefix = condense( CONV string( ls_cfg-bms_path_prefix ) ).

    IF rv_prefix IS NOT INITIAL
       AND substring( val = rv_prefix off = strlen( rv_prefix ) - 1 ) = '/'.
      rv_prefix = substring( val = rv_prefix len = strlen( rv_prefix ) - 1 ).
    ENDIF.

  ENDMETHOD.
