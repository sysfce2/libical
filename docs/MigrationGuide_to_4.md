# Migrating to version 4

A guide to help developers port their code from libical v3.x to libical 4.0.

## Conditional compilation

To continue supporting the 3.0 version you can use conditional compilation, like so:

```C
     #if ICAL_CHECK_VERSION(4,0,0)
     <...new code for the libical 4.0 version ...>
     #else
     <...old code for the libical 3.0 version ...>
     #endif
```

you can handle code that no longer exists in 4.0 with:

```C
     #if !ICAL_CHECK_VERSION(4,0,0)
     <...old code for the libical 3.0 version ...>
     #endif
```

## C library

### Modified functions

* `icalrecurrencetype_from_string()` was replaced by `icalrecurrencetype_new_from_string()`,
   which returns a `struct icalrecurrencetype *` rather than a `struct icalrecurrencetype`.
* The following functions now take arguments of type `struct icalrecurrencetype *` rather than
  `struct icalrecurrencetype`:
  * `icalproperty_new_rrule()`
  * `icalproperty_get_rrule()`
  * `icalproperty_set_rrule()`
  * `icalproperty_vanew_rrule()`
  * `icalproperty_new_exrule()`
  * `icalproperty_set_exrule()`
  * `icalproperty_get_exrule()`
  * `icalproperty_vanew_exrule()`
  * `icalrecur_iterator_new()`
  * `icalvalue_new_recur()`
  * `icalvalue_set_recur()`
  * `icalvalue_get_recur()`

* The following functions now return a value of type `struct icalrecurrencetype *` rather than
  `struct icalrecurrencetype`:
  * `icalproperty_get_rrule()`
  * `icalproperty_get_exrule()`
  * `icalvalue_get_recur()`

### New functions

The following functions have been added:

* `icalarray_set_element_at()`
* `icalrecurrencetype_new()`
* `icalrecurrencetype_ref()`
* `icalrecurrencetype_unref()`
* `icalrecurrencetype_clone()`
* `icalrecurrencetype_encode_day()`
* `icalrecurrencetype_encode_month()`
* `icaltzutil_set_zone_directory()`
* `icalcomponent_clone()`
* `icalproperty_clone()`
* `icalparameter_clone()`
* `icalparameter_kind_value_kind()`
* `icalparameter_is_multivalued()`
* `icalparameter_decode_value()`
* `icalvalue_clone()`
* `icalcluster_clone()`
* `icalrecur_iterator_prev()`
* `icalrecur_resize_by()`
* `icalrecurrencetype_new()`
* `icalrecurrencetype_ref()`
* `icalrecurrencetype_unref()`
* `icalrecurrencetype_clone()`
* `icalrecurrencetype_from_string()`
* `icalcomponent_set_x_name()`
* `icalcomponent_get_x_name()`
* `icalcomponent_get_component_name()`
* `icalcomponent_get_component_name_r()`
* `ical_set_invalid_rrule_handling_setting()`
* `ical_get_invalid_rrule_handling_setting()`
* `icalparser_get_ctrl()`
* `icalparser_set_ctrl()`
* and the new functions for the `icalstrarray` and `icalenumarray` data types

### Removed functions

* `icalrecurrencetype_clear()` has been removed.
* These deprecated functions have been removed:
  * `caldat()`
  * `juldat()`
  * `icalcomponent_new_clone()`
  * `icalparameter_new_clone()`
  * `icalproperty_new_clone()`
  * `icalvalue_new_clone()`
  * `icalcluster_new_clone()`

* No longer publicly visible functions:
  * `icaltzutil_fetch_timezone()`
  * `icalrecurrencetype_clear()`

### Added data types

* These data types have been added:
  * icalstrarray - for manipulating an array of strings
  * icalenumarray_element - structure to hold a generic enum value
  * icalenumarray - for manipulating an array of enum elements

### Removed data types

* These data structures have been removed (as they were never used):
  * struct icaltimezonetype
  * struct icaltimezonephase

### Migrating from 3.0 to 4.0

### bool return values

A number of function signatures have been changed to use 'bool' rather than 'int' types.

This is implemented using the C99 standards compliant <stdbool.h> header.

### Clone functions

Replace all `ical*_new_clone()` function calls with `ical*_clone()` .
ie, use `icalcomponent_clone()` rather then `icalcomponent_new_clone()`.

### `icalrecurrencetype` now passed by reference

The way `struct icalrecurrencetype` is passed between functions has been changed. While it was
usually passed by value in 3.0, it is now passed by reference. A reference counting mechanism is
applied that takes care of de-allocating an instance as soon as the reference counter goes to 0.

Code like this in libical 3.0:

```C
    struct icalrecurrencetype recur;

    icalrecurrencetype_clear(&recur);

    // Work with the object
```

changes to this in libical 4.0:

```C
    struct icalrecurrencetype *recur;

    // allocate
    recur = icalrecurrencetype_new();
    if (recur) {

        // Work with the object

        // deallocate
        icalrecurrencetype_unref(recur);
    } else {
        // out of memory error handling
    }
```

### `icalgeotype` now uses character strings rather than doubles

The members of `struct icalgeotype` for latitude ('lat`) and longitude ('lon`) have been changed
to use ICAL_GEO_LEN long character strings rather than the double type.

This means that simple assignments in 3.0 must be replaced by string copies.

```C
     geo.lat = 0.0;
     geo.lon = 10.0;
```

becomes

```C
     strncpy(geo.lat, "0.0", ICAL_GEO_LEN-1);
     strncpy(geo.lon, "10.0", ICAL_GEO_LEN-1);
```

and

```C
    double lat = geo.lat;
    double lon = geo.lon;
```

becomes

```C
    double lat, lon;
    sscanf(geo.lat, "%lf", &lat);
    sscanf(geo.lon, "%lf", &lon);
```

### Working with `icalvalue` and `icalproperty`

Code like this in libical 3.0:

```C
    icalvalue *recur_value = ...;
    struct icalrecurrencetype recur = icalvalue_get_recur(recur_value);

    // Work with the object
```

changes to this in libical 4.0:

```C
    icalvalue *recur_value = ...;
    struct icalrecurrencetype *recur = icalvalue_get_recur(recur_value);

    // Work with the object
    // No need to unref
```

### Multi-valued parameters

Support for these multi-valued parameters is added in libical 4.0.

* DELEGATED-FROM (RFC 5545)
* DELEGATED-TO (RFC 5545)
* MEMBER (RFC 5545)
* DISPLAY (RFC 7986)
* FEATURE (RFC 7986)

You can access the 'nth' value for such parameters using the new "_nth" functions.

For example, to access the first delegated-to attendee use

```c
param = icalproperty_get_first_parameter(prop, ICAL_DELEGATEDTO_PARAMETER);
icalparameter_get_delegatedto_nth(param, 0)
```

## C++ library

### Modified methods

* The following methods now take arguments of type `struct icalrecurrencetype *` rather than `const
  struct icalrecurrencetype &`:
  * `ICalValue.set_recur()`
  * `ICalProperty.set_exrule()`
  * `ICalProperty.set_rrule()`

* The following methods now returns a value of type `struct icalrecurrencetype *` rather than
  `struct icalrecurrencetype`:
  * `ICalValue.get_recur()`
  * `ICalProperty.get_exrule()`
  * `ICalProperty.get_rrule()`

### `icalrecurrencetype.by_xxx` static arrays replaced by dynamically allocated ones

I.e. memory `short by_hour[ICAL_BY_DAY_SIZE]` etc. are replaced by

```c
typedef struct
{
  short *data;
  short size;
} icalrecurrence_by_data;

struct icalrecurrencetype {
  ...
  icalrecurrence_by_data by[ICAL_BY_NUM_PARTS];
}
```

Memory is allocated in the required size using the new `icalrecur_resize_by()` function. It is
automatically freed together with the containing `icalrecurrencetype`. As the size of the array is
stored explicitly, no termination of the array with special value `ICAL_RECURRENCE_ARRAY_MAX` is
required anymore.  The array is iterated by comparing the iterator to the `size` member value.

### Migrating `icalrecurrencetype.by_xxx` static arrays usage from 3.0 to 4.0

Code like this in libical 3.0:

```C
    icalrecurrencetype recur;
    ...
    recur.by_hour[0] = 12;
    recur.by_hour[1] = ICAL_RECURRENCE_ARRAY_MAX;
```

changes to something like this in libical 4.0:

```C
    icalrecurrencetype *recur;
    ...
    if (!icalrecur_resize_by(&recur->by[ICAL_BY_HOUR], 1)) {
      // allocation failed
      // error handling
    } else {
      recur.by[ICAL_BY_HOUR].data[0] = 12;
    }
```

## GLib/Python bindings - changed `ICalGLib.Recurrence.*_by_*` methods

`i_cal_recurrence_*_by_xxx*` methods have been replaced by more generic versions that take the 'by'
type (day, month, ...) as a parameter.

### Migrating `ICalGLib.Recurrence.*_by_*` methods from 3.0 to 4.0

Code like this in libical 3.0:

```python
    recurrence.set_by_second(0,
    recurrence.get_by_second(0) + 1)
```

changes to something like this in libical 4.0:

```python
    recurrence.set_by(ICalGLib.RecurrenceByRule.BY_SECOND, 0,
    recurrence.get_by(ICalGLib.RecurrenceByRule.BY_SECOND, 0) + 1)
```
