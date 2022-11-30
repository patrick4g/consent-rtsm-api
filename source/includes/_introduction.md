# Introduction

This document describes two separate APIs:

1. <b>`RTSM API`</b> (Thread -> 4G):  This is hosted by 4G Clinical.  It is used to:
   1. query patient information from RTSM
   2. notify RTSM of new eConsent patients events
   3. notify RTSM of certain participant status change events
2. <b>`ECONSENT API`</b> (4G -> Thread): This is hosted by Thread.  It provides endpoints to:
   1. query participant information from eConsent
   2. notify eConsent of certain RTSM status changes (incl. Screening, Randomization and others as specified)

Both APIs use REST with JSON payloads.
