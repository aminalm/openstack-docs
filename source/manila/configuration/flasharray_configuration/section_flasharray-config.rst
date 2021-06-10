FlashArray Configuration
========================

.. _manila_flasharray_prerequisites:

FlashArray Prerequisites
------------------------
The prerequisites for Pure Storage FlashArray are:

- The FlashArray must have the FA-Files feature enabled.

- The driver requires a storage admin level account to use with the
  OpenStack to manage the FlashArray. Optionally, you can use an
  existing account with a minimum of storage admin privilege for integration.

- The Management VIP address is required to properly configure the FlashBlade driver.

- The File VIP address is required to properly configure the FlashBlade driver.

Management VIP
--------------
Administrators and hosts can access the FlashArray using virtual IP addresses.
The Management Virtual IP (MVIP) enables cluster management through a 1GbE connection.

File VIP
--------
Users and hosts can access the FlashArray shares using virtual IP interfaces.
A File Virtual IP (VIF) enables dataplane access through bound interfaces

Admin Account
-------------

Pure Storage recommends creating an unique user account for OpenStack
to help audit API requests from each OpenStack. This account may be
local or integrated into an Active Directory, LDAP, or similar, account
management system.
