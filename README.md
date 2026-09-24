    DATA(lv_uri) = lo_http->request->get_header_field( '~request_uri' ).
    IF lv_uri IS NOT INITIAL
       AND substring( val = lv_uri off = strlen( lv_uri ) - 1 ) = '/'.
      lv_uri = substring( val = lv_uri len = strlen( lv_uri ) - 1 ).
    ENDIF.
