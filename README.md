  METHOD post_bms_order.

    CLEAR: ev_http_status, ev_response.

    DATA lo_http TYPE REF TO if_http_client.

    cl_http_client=>create_by_destination(
      EXPORTING
        destination              = iv_destination
      IMPORTING
        client                   = lo_http
      EXCEPTIONS
        argument_not_found       = 1
        destination_not_found    = 2
        destination_no_authority = 3
        plugin_not_active        = 4
        internal_error           = 5
        OTHERS                   = 6 ).

    IF sy-subrc <> 0.
      ev_response = |Destination { iv_destination } nicht nutzbar (subrc { sy-subrc })|.
      RETURN.
    ENDIF.

    " The path prefix (/BmsApiSapTest in TI4) is maintained in SM59 on the
    " destination, not in the coding. CREATE_BY_DESTINATION puts it into
    " the request URI; SET_REQUEST_URI would overwrite it, so the prefix
    " is read back first and the endpoint appended.
    DATA(lv_uri) = lo_http->request->get_header_field( '~request_uri' ).
    IF lv_uri IS NOT INITIAL
       AND substring( val = lv_uri off = strlen( lv_uri ) - 1 ) = '/'.
      lv_uri = substring( val = lv_uri len = strlen( lv_uri ) - 1 ).
    ENDIF.

    cl_http_utility=>set_request_uri(
      request = lo_http->request
      uri     = |{ lv_uri }/api/container/create-order-halle| ).

    lo_http->request->set_method( if_http_request=>co_request_method_post ).
    lo_http->request->set_content_type( 'application/json' ).
    lo_http->request->set_header_field( name  = 'Authorization'
                                        value = iv_bearer_token ).
    lo_http->request->set_cdata( iv_json ).

    lo_http->send( EXCEPTIONS OTHERS = 4 ).
    IF sy-subrc <> 0.
      lo_http->get_last_error( IMPORTING message = ev_response ).
      lo_http->close( ).
      RETURN.
    ENDIF.

    lo_http->receive( EXCEPTIONS OTHERS = 4 ).
    IF sy-subrc <> 0.
      lo_http->get_last_error( IMPORTING message = ev_response ).
      lo_http->close( ).
      RETURN.
    ENDIF.

    lo_http->response->get_status( IMPORTING code = ev_http_status ).
    ev_response = lo_http->response->get_cdata( ).
    lo_http->close( ).

  ENDMETHOD.
