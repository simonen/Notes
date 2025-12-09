
https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/security_hardening/auditing-the-system_security-hardening#starting-the-audit-service_auditing-the-system

Record activity in a directory

```
auditctl -w /data/sambashare -p war -k file_access
```

```
ausearch -k file_access
```

