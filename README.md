LOCATE
======

This document describes a system which can be used to accurately
determine the location of a person who calls an emergency number. The
key differentiator here is that this system does not require the use of
a data network, and requires minimal additional infrastructure to
support it.

Motivation
==========

I was watching [John Oliver’s segment on
911](https://www.youtube.com/watch?v=A-XlyB_QQYs) the other day, and one
of the major problems that he talked about was that emergency responders
do not know the location of callers, or do not know the location
accurately enough.

The basic problem here is that while services like Uber can use their
own apps to get accurate location information, emergency services are
limited to using location information provided by telecom operators,
which usually relies on cell triangulation to get a rough location.

There are initiatives to do better, for example, NG9-1-1, which rely on
data networks, and will likely require major infrastructure upgrades to
implement. A problem with all of these “enhanced” emergency call
services is that everybody is coming up with their own system; USA and
Canada have one proposal, while Europe has another.

This was my motivation to come up with a system that:

-   can be implemented right now and without major infrastructure
    upgrades,
-   should be able to work anywhere in the world where cell phones can
    be used,
-   should be seamless for the user, and
-   does not need a data network to be available in order to function.
-   does only one thing: **send an accurate location of a user who calls
    an emergency number**.

This system is not intended to replace any of the enhanced emergency
call service proposals; rather, it can complement those systems, because
it has a singular purpose.

The System
==========

In one line: when a user calls an emergency number, send a text message
(SMS) to that same number with location information, and repeat if a
more accurate location is available subsequently.

Making emergency calls from a cellular phone
--------------------------------------------

It is up to the phone to determine which numbers are classified as
emergency numbers, as most, if not all, phones are programmed with lists
of emergency numbers.

When a user dials an emergency number and the call connects: (time = t0)

1.  Start gathering location data, using GPS, Wi-Fi/Bluetooth, or any
    other means available to the phone.
2.  30 seconds later: (time = t1) if a location is available, send the
    current location using the format described in “Sending location.”
    If a location is not available, send the current location as soon as
    it becomes available.
3.  Keep monitoring the location until t0 + 15 minutes (time = t2). If
    the uncertainty in location (as a radius around a point) has
    decreased by at least p0 = 50% of the previous value; or if the
    uncertainty is the best (most accurate) that it can possibly be (as
    determined by the location services on the phone), resend the
    current location.
    *   Within this monitoring period, if the location has changed
        (distance between the last location and the current one) by more
        than p1 = 5 times the uncertainty value, resend the location.
        This is so that emergency services know if the user is moving.
    *   Do not send an update more frequently than once every p2 =
        60 seconds.

The phone may allow the user to turn off this system, with appropriate
warnings; with the automatic system turned off, the phone may present a
user interface during or after an emergency call to manually send the
location using the same system.

If the system is manually triggered, the same algorithm applies, with t0
= the time when the user expresses their intent to send their location.

Accessibility
-------------

The same protocol may also be used for text messages that a user types
and sends to an emergency number. This may be useful for users who do
not have the ability to use voice calls, but can communicate using text
messaging.

In this case, t0 is the time at which a user-written text message is
sent to the cellular network. The phone will simply send additional text
messages with the location information, as described above.

The phone must not send the location in the same message that the user
has typed.

Sending location
----------------

The location is sent using a text message to the emergency number that
was called.

The text message must contain the current location, formatted as a
‘geo:’ URI as described in [RFC
5870](https://tools.ietf.org/html/rfc5870). The URI must include the ‘u’
(uncertainty) parameter, in addition to the spatial coordinates.

To prevent trivial spoofing, the location may be sent in the UDH part of
the SMS (tag to be determined.)

Telecom infrastructure
----------------------

Telecom operators must accept and deliver text messages sent to
emergency numbers. Since it is possible to easily spoof sender
identification in text messages when connected to a telco network,
telecom operators should verify that messages sent to emergency numbers
are actually originating from the phone they claim to be originating
from.

Telecom operators must ensure that both calls and messages to emergency
numbers are routed to the same organization, especially in roaming
scenarios.

Telecom operators may choose to make calls and/or messages to emergency
numbers free of charge for the phone service user. They may also choose
to allow these calls and messages when a phone does not have a SIM, or
has a SIM which is not valid on their network. These policies should be
in line with their existing emergency call policies.

Emergency response infrastructure
---------------------------------

Emergency services will need to set up the infrastructure necessary to
be able to receive text messages sent to emergency numbers.

The emergency service must:

1.  Correlate text messages received with calls received from the
    same number.
2.  Filter messages that have location information from messages that
    do not. Valid location messages will only have the geo URI, either
    in the body or the UDH, and no other text content.
3.  Allow geo URIs with more parameters than specified in the protocol,
    as long as the location and uncertainty are present. Phones may
    choose to append additional diagnostic information in the URIs; or
    the protocol may be extended later to allow more parameters.
4.  Update location information as it is received in their database
    records associated with that particular call.
5.  Store all location records received.
6.  Determine if the caller is moving by looking at the
    location records. If the distance between two records is more than
    the uncertainty values, the caller is likely to be moving.
7.  Further forward this location information to the
    responders assigned. The service may use whatever system they have
    in place already to provide location updates as they come in.

Fine-tuning
-----------

There are six parameters in the system (t0, t1, t2, p0, p1, p2) which
can be fine-tuned (based on actual usage, perhaps.)

Caveats
-------

While I have stated that the system does not need a data network to
function, various location technologies do need a data network, for
example, A-GPS and Wi-Fi/Bluetooth micro-location. However, most phones
should be able to provide a location more accurate than one from cell
triangulation, even without Internet access: as long as a GPS signal is
available, the location can be determined within a few minutes.

The definition of “major infrastructure upgrades” is arbitrary.

Contact
=======

You can contact the author via email at <nikhil@nikhildabas.com>.

Copyright notice
================

Copyright © 2016 Nikhil Dabas. All rights reserved.
