Multi-instance Tomcat
=======================
- `CATALINE_HOME`: shared static binaries
- `CATALINE_BASE`: instance specific conf etc.

Manager App
=================
`CATALINE_BASE/conf/tomcat-user.xml`

First figure out what roles is defined by manager app:
```xml
$ grep -i role webapps/manager/WEB-INF/web.xml 
  <!-- NOTE:  None of these roles are present in the default users file -->
       <role-name>manager-gui</role-name>
       <role-name>manager-script</role-name>
       <role-name>manager-jmx</role-name>
       <role-name>manager-gui</role-name>
       <role-name>manager-script</role-name>
       <role-name>manager-jmx</role-name>
       <role-name>manager-status</role-name>
  <!-- Security roles referenced by this web application -->
  <security-role>
      The role that is required to access the HTML Manager pages
    <role-name>manager-gui</role-name>
  </security-role>
  <security-role>
      The role that is required to access the text Manager pages
    <role-name>manager-script</role-name>
  </security-role>
  <security-role>
      The role that is required to access the HTML JMX Proxy
    <role-name>manager-jmx</role-name>
  </security-role>
  <security-role>
      The role that is required to access to the Manager Status pages
    <role-name>manager-status</role-name>
  </security-role>

```

Then add a user with certain role:
```xml
$ vim conf/tomcat-users.xml
...
    <user username="manager_admin" password="admin" roles="manager-gui" />
</tomcat-users>
```

Host Manager App
===========
Find roles referenced in `webapps/host-manager/WEB-INF/web.xml`.

```
  <!-- Security roles referenced by this web application -->
  <security-role>
    <description>
      The role that is required to log in to the Host Manager Application HTML
      interface
    </description>
    <role-name>admin-gui</role-name>
  </security-role>
  <security-role>
    <description>
      The role that is required to log in to the Host Manager Application text
      interface
    </description>
    <role-name>admin-script</role-name>
  </security-role>
```

Apache Tomcat default globals settings (servlets / mime mappings / welcome-files)
===================
For stock tomcat release tarball, 2 defaults servlets `default` and `jsp` are enabled in `CATALINE_BASE/conf/web.xml`.

- `default` is the fallback servlet for all reqeusts with path that does not match any other filters or servlets. It's behavior is to supply the static file if the request path matches the filesystem hierarchy, or return 404 otherwise.
- `jsp` is used to support jsp. It captures all requests with path ended with `.jsp` or `.jspx`.

`session-timeout` was set to 30 minute:

```xml
    <session-config>
        <session-timeout>30</session-timeout>
    </session-config>
```
Huge amount of mime-mapping was defined for common file extensions.

are defined with the following priority

```xml
    <welcome-file-list>
        <welcome-file>index.html</welcome-file>
        <welcome-file>index.htm</welcome-file>
        <welcome-file>index.jsp</welcome-file>
    </welcome-file-list>
```

