  METHOD get_bms_bearer_token.

    CLEAR: ev_token, ev_error.

    DATA lo_http  TYPE REF TO if_http_client.
    DATA lv_code  TYPE i.
    DATA lv_subrc TYPE sy-subrc.

    " Serialise rather than concatenate — a password containing " or \
    " would otherwise produce invalid JSON.
    DATA: BEGIN OF ls_auth_req,
            username TYPE string,
            password TYPE string,
          END OF ls_auth_req.

    ls_auth_req-username = condense( CONV string( iv_username ) ).
    ls_auth_req-password = condense( CONV string( iv_password ) ).

    DATA(lv_auth_json) = /ui2/cl_json=>serialize(
      data        = ls_auth_req
      compress    = abap_false
      pretty_name = /ui2/cl_json=>pretty_mode-camel_case ).

    CLEAR ls_auth_req.

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
      lv_subrc = sy-subrc.
      MESSAGE e054(z_msg_svr_tour_ext) WITH iv_destination lv_subrc INTO ev_error.
      RETURN.
    ENDIF.

    " Path prefix from SM59, see POST_BMS_ORDER
    DATA(lv_uri) = lo_http->request->get_header_field( '~request_uri' ).
    IF lv_uri IS NOT INITIAL
       AND substring( val = lv_uri off = strlen( lv_uri ) - 1 ) = '/'.
      lv_uri = substring( val = lv_uri len = strlen( lv_uri ) - 1 ).
    ENDIF.

    cl_http_utility=>set_request_uri(
      request = lo_http->request
      uri     = |{ lv_uri }/api/user/authenticate/user| ).

    lo_http->request->set_method( if_http_request=>co_request_method_post ).
    lo_http->request->set_content_type( 'application/json' ).
    lo_http->request->set_cdata( lv_auth_json ).
    CLEAR lv_auth_json.

    lo_http->send( EXCEPTIONS OTHERS = 4 ).
    IF sy-subrc <> 0.
      lo_http->get_last_error( IMPORTING message = ev_error ).
      lo_http->close( ).
      RETURN.
    ENDIF.

    lo_http->receive( EXCEPTIONS OTHERS = 4 ).
    IF sy-subrc <> 0.
      lo_http->get_last_error( IMPORTING message = ev_error ).
      lo_http->close( ).
      RETURN.
    ENDIF.

    lo_http->response->get_status( IMPORTING code = lv_code ).
    DATA(lv_body) = lo_http->response->get_cdata( ).
    lo_http->close( ).

    IF lv_code <> 200.
      MESSAGE e055(z_msg_svr_tour_ext) WITH lv_code lv_body INTO ev_error.
      RETURN.
    ENDIF.

    TYPES: BEGIN OF ty_auth_data,
             access_token  TYPE string,
             refresh_token TYPE string,
           END OF ty_auth_data.
    TYPES: BEGIN OF ty_auth_response,
             data TYPE ty_auth_data,
           END OF ty_auth_response.

    DATA ls_auth TYPE ty_auth_response.

    /ui2/cl_json=>deserialize(
      EXPORTING json        = lv_body
                pretty_name = /ui2/cl_json=>pretty_mode-camel_case
      CHANGING  data        = ls_auth ).

    IF ls_auth-data-access_token IS INITIAL.
      MESSAGE e056(z_msg_svr_tour_ext) INTO ev_error.
      RETURN.
    ENDIF.

    ev_token = |Bearer { ls_auth-data-access_token }|.

  ENDMETHOD.
