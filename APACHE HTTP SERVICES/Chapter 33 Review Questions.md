
1. Which directive should you use for a directory that has contents that should
	never be accessed by the Apache web server?
	**Require All Denied**
2. Which Boolean should be set to ease the SELinux rules for Apache a bit?
	**httpd_unified on**
3. Which SELinux context type do you need to set on a directory that is used as
	the DocumentRoot?
	**httpd_sys_content_t**
4. In which directory would you typically find the TLS certificate file and the
	**TLS** key?
	**/etc/pki/tls/**
5.  Which parameter is used to identify the private key that a TLS secured virtual
	host should use?
	**SSLCertificateKeyFile**
6.  What is the default configuration file for configuring TLS secured web
	servers?
	**/etc/httpd/conf.d/ssl.conf**
7. Which command enables you to create a TLS certificate?
	**genkey**, openssl
8. How do you configure Apache to run the Python script /opt/webapp/app.py?
	**wsgi_mod**, **WSGIScriptAlias** /webapp/ /opt/webapp/app.py
9. What is wrong with using the command htpasswd -c /etc/httpd/htpasswd
	lisa if you have already created some Apache users?
	the file will be overwritten with the latest user
10. How do you configure a directory “secret” that is accessible for authenticated
	users only?
	<Directory /secret>
		AuthType Basic
		AuthUserFile /htpasswdfile
		Require valid-user
	\</Directory>
	