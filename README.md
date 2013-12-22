Z ABAP Remote Comparison (ZARC)
====
ZARC is a set o classes that helps you compare one or more ABAP programs, tables, data elements, domains, etc among multiple SAP systems that are set under the same transport domain. It's based on the Version Management functionality of the ABAP Workbench, but without the need to check every single ABAP object manually.

### How to use ZARC

The first thing you need to do is install ZARC on you SAP system. To do so, create an include using SE38 named ZARC and paste all ZARC.TXT source on it, then activate it. After that, you can create a new report on SE38 named ZARC_EXAMPLE, that starts with the following lines:

```ABAP
REPORT zarc_example.

* This type-pool is used by ZARC basically to compare flags to ABAP_TRUE and ABAP_FALSE.
TYPE-POOLS abap.

* This is the include we just created.
INCLUDE zarc. " this contains all ZARC classes needed
```

Then, you at your START-OF-SELECTION event, you can declare these variables bellow.

```ABAP
START-OF-SELECTION.

* ZARC main object, used to instantiate ZARC from the object factory
  DATA : oref TYPE REF TO if_zarc_remote_compare.

* ZARC exception objects, here used only to prevent you from having a short dump :)
  DATA : ex_ofe TYPE REF TO cx_zarc_objfac_exception,
         ex_ce TYPE REF TO cx_zarc_compare_exception,
         ex_nie TYPE REF TO cx_zarc_notimplemented_ex.

* ZARC changeflag
  DATA changed TYPE flag.
```
And here comes the full example program, applying all the steps you read above.

```ABAP
REPORT zarc_example.

TYPE-POOLS abap.

INCLUDE zarc. " this contains all ZARC classes needed

START-OF-SELECTION.

  BREAK-POINT.

* ZARC main object
  DATA : oref TYPE REF TO if_zarc_remote_compare.

* ZARC exception objects
  DATA : ex_ofe TYPE REF TO cx_zarc_objfac_exception,
         ex_ce TYPE REF TO cx_zarc_compare_exception,
         ex_nie TYPE REF TO cx_zarc_notimplemented_ex.

* ZARC changeflag
  DATA changed TYPE flag.

  TRY.

*     Step 1: get an instance of IF_ZARC_REMOTE_COMPARE based on the OBJTYPE parameter
      oref = cl_zarc_remote_compare=>factory( objtype = 'REPS' objname = 'RSHOWTIM' ).

*     Step 2: add the systems you want to use for version comparison (you can compare QAS and PRD even if you're runing it on DEV, yay!)
      oref->add_system_for_compare( 'DEV' ).
      oref->add_system_for_compare( 'QAS' ).

*     Step 3: execute the comparison, so that you can receive the true/false changeflag
      changed = oref->run_comparison( ).

      IF changed EQ abap_true.
        MESSAGE 'The object does contains changes.' TYPE 'I'.
*       Go to standard SAP Version Management diff report.
        oref->display_diff( ).
      ELSE.
        MESSAGE 'The object is the same.' TYPE 'I'.
      ENDIF.

    CATCH cx_zarc_objfac_exception INTO ex_ofe.
*     This exception is thrown when the Step 1 fails. It can happen when there's no CL_ZARC_REMOTE_COMPARE_XXXX definition to be instantiated, where XXXX equals OBJTYPE parameter.
    CATCH cx_zarc_compare_exception INTO ex_ce.
*     This exception is thrown when the Step 3 fails. This can happen when you try to run the comparison before add at least two systems on Step 2.
    CATCH cx_zarc_notimplemented_ex INTO ex_nie.
*     This exception is thrown when the Step 3 fails. It means that something inside ZARC isn't quite ready yet. But the good news is that you can help us! :)
  ENDTRY.

  BREAK-POINT.
```

### Feedback and help wanted
Yes, I've put it here under the GPL V2 license so that we can improve it together. There are some classes that still need to be created, and functionality that needs to be implemented. You're welcome!
