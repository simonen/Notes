Pluggable Authentication Module

[Chapter 5 PAM Apress - Pro Linux System Administration 2nd]

`/etc/pam.d/`

`so`: shared object

PAMs control how and when users can interact with a host. It is a hierarchy of authentication and authorization checks. These checks are stacked together. Checks can be required or optional.

- `/etc/pam.d/passwd` is a **service configuration files** - the service shares the same name as the application it is designed to authenticate.
- `/etc/pam.d/system-auth` - generated when installing the host, updated with `authconfig`. CentOS. On Ubuntu, the same role is performed by four files: common-auth, common-password, common-session, common-account

Management groups in PAM Service configuration files have four possible directives:
* `auth | required | /usr/local/pamlib/pam_local.so | nullok try_first_pass` 

Management groups
* **auth**: These modules perform user authentication, checking a password
* **account**: Handles account verification tasks - confirming that the account is not locked or if only root can perform the action
* **password**: Setting passwords - checking if the password is strong enough for example
* **session**: check, manage and configure user sessions

Control flags:
* `required`: a required module must succeed for authentication to succeed
* `requisite`: if a requisite module fails, authentication immediately fails
* `sufficient`: authentication immediately succeeds if module is successful
* `optional`: authentication is not impacted by module success or failure

- `pam_unix.so`: PAM module. If no path is specified, **/lib/security** is assumed
- `nullok try_first_pass`: this directive contains arguments that are passed to the pam module
- `try_first_pass`: this argument tells the module to see if a password has already been received by the module, and if so, to use that pass for auth
- `nullok`: this argument tells the module that a blank pass is ok

Most modules will ignore invalid or incorrect arguments passed to them

The `include` function allows to include one PAM file in another

POSIX 

Portable Operating System Interface

`possixAccount`: 

- `posixAccount` is an LDAP object class that represents a user account in a POSIX-compliant operating system environment.
- It is defined in the RFC 2307bis standard.
- Attributes associated with `posixAccount` include POSIX-specific attributes such as UID (User ID), GID (Group ID), home directory, and login shell.
- `posixAccount` is commonly used in environments where LDAP is used as a central user directory service for UNIX or Linux systems, as it provides attributes required for POSIX account management.
- Unlike `inetOrgPerson`, `posixAccount` focuses on attributes relevant to user authentication and account management in POSIX-compliant systems.

**inetOrgPerson**:
    
- `inetOrgPerson` is a widely-used LDAP object class that represents a person or entity within an organization.
    - It is defined in the Internet Engineering Task Force (IETF) RFC 2798.
    - Attributes associated with `inetOrgPerson` typically include personal information such as name, email address, phone number, and organizational details.
 - `inetOrgPerson` is often used in general-purpose directory services to store information about users, employees, or individuals within an organization.
    - While `inetOrgPerson` provides attributes for basic user information, it does not include attributes specific to POSIX (Portable Operating System Interface) accounts, such as UID (User ID) and GID (Group ID).

