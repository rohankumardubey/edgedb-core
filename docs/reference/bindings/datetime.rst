.. _ref_bindings_datetime:

==================
Date/Time Handling
==================

EdgeDB has 6 types related to date and time handling:

* :eql:type:`datetime` (:ref:`binary format <ref_protocol_fmt_datetime>`)
* :eql:type:`duration` (:ref:`binary format <ref_protocol_fmt_duration>`)
* :eql:type:`cal::local_datetime`
  (:ref:`binary format <ref_protocol_fmt_local_datetime>`)
* :eql:type:`cal::local_date`
  (:ref:`binary format <ref_protocol_fmt_local_date>`)
* :eql:type:`cal::relative_duration`
  (:ref:`binary format <ref_protocol_fmt_relative_duration>`)
* :eql:type:`cal::date_duration`
  (:ref:`binary format <ref_protocol_fmt_date_duration>`)

We generally map datetime values to a compatible native type in the binding's
language when that native type meets all these criteria:

* The type is in the language's standard library
* The type has sufficient range (EdgeDB timestamps range from year 1 to 9999)
* The type has sufficient precision (at least microseconds)

If any of the above criteria are *not* met, we usually provide a custom type
in the client library that can be converted to a language-native type or a 
type provided by a third-party library.

.. note::

    Strict adherence to this guideline doesn't make sense in some languages.
    If you think this applies to your binding, either discuss it with us or
    use your best judgment.


Precision
=========

:eql:type:`datetime`, :eql:type:`duration`, :eql:type:`cal::local_datetime` and
:eql:type:`cal::relative_duration` all have precision of **1 millisecond** in
EdgeDB. If the language-native type is more precise (e.g., nanosecond
precision), the client library must round the value when encoding it for
storage in the database. We use a **round to the nearest even** strategy for
that operation. Here are some examples of timestamps with high precision and
how they are stored in the database::

    2022-02-24T05:43:03.123456789Z → 2022-02-24T05:43:03.123457Z

    2022-02-24T05:43:03.000002345Z → 2022-02-24T05:43:03.000002Z
    2022-02-24T05:43:03.000002500Z → 2022-02-24T05:43:03.000002Z
    2022-02-24T05:43:03.000002501Z → 2022-02-24T05:43:03.000003Z
    2022-02-24T05:43:03.000002499Z → 2022-02-24T05:43:03.000002Z

    2022-02-24T05:43:03.000001234Z → 2022-02-24T05:43:03.000001Z
    2022-02-24T05:43:03.000001500Z → 2022-02-24T05:43:03.000002Z
    2022-02-24T05:43:03.000001501Z → 2022-02-24T05:43:03.000002Z
    2022-02-24T05:43:03.000001499Z → 2022-02-24T05:43:03.000001Z

.. note::

    As described in our :ref:`datetime protocol documentation
    <ref_protocol_fmt_datetime>`, the value is encoded as a *signed*
    microseconds delta since a fixed time. Some care must be taken when
    rounding negative microsecond values. See `tests for Rust implementation`_
    for a good set of test cases.

EdgeDB client libraries round to the nearest even for all operations they
perform that require rounding, in particular:

* Encoding timestamps *and* time deltas (see the :ref:`list of datetime types
  <ref_bindings_datetime>`) to the binary format if precision of the native
  type is greater than microseconds.
* Decoding timestamps *and* time deltas from the binary format if precision
  of the native type is less than microseconds (applies to JavaScript for
  example)
* Converting from EdgeDB-specific type (if there is one) to a native type and
  back when those types differ in their precision
* Parsing a string to an EdgeDB-specific type (Implementation is not required,
  but if it *is* implemented, it must use this rounding strategy.)

.. lint-off

.. _tests for Rust implementation: https://github.com/edgedb/edgedb-rust/tree/master/edgedb-protocol/tests/datetime_chrono.rs

.. lint-on
