    DATA(lv_uri) = lo_http->request->get_header_field( '~request_uri' ).
    IF lv_uri IS NOT INITIAL
       AND substring( val = lv_uri off = strlen( lv_uri ) - 1 ) = '/'.
      lv_uri = substring( val = lv_uri len = strlen( lv_uri ) - 1 ).
    ENDIF.


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
