## SPNEGO extension

This is a packaging project that re-package [spnego implementation](https://github.com/gatein/gatein-sso/tree/master/spnegosso) into eXo Addons packaging format.

### How to install

```
./addon.sh install exo-spnego
```

### How to configure SPNEGO SSO for eXo Platform

The following guideline only guide you how to configure spnego sso for eXo Platform.
For more information about SPNEGO and how to setup environments, you should read SPNEGO section at [eXo Platform Documentation](http://docs.exoplatform.com/public/index.jsp?topic=%2FPLF40%2FSingle_Sign_On-SPNEGO.html&cp=1_7_4_7_3)


##### eXo Platform Tomcat

1. Append security-domain `spnego-server` into `$PLATFORM_HOME/conf/jaas.conf`
```
spnego-server {
	com.sun.security.auth.module.Krb5LoginModule required
	storeKey=true
	doNotPrompt=true
	useKeyTab=true
	keyTab="/etc/krb5.keytab"
	principal="HTTP/server.example.com@EXAMPLE.COM"
	useFirstPass=true
	debug=true
	isInitiator=false;
};
```

2. Configure SSO in `$PLATFORM_HOME/gatein/conf/configuration.properties`:
```
gatein.sso.enabled=true
gatein.sso.filter.spnego.enabled=true
gatein.sso.callback.enabled=false
gatein.sso.skip.jsp.redirection=false
gatein.sso.login.module.enabled=true
gatein.sso.login.module.class=org.gatein.security.sso.spnego.SPNEGOSSOLoginModule
gatein.sso.filter.login.sso.url=/@@portal.container.name@@/spnegosso
gatein.sso.filter.initiatelogin.enabled=false
gatein.sso.valve.enabled=false
gatein.sso.filter.logout.enabled=false
```

3. Set `CATALINA_OPTS` in `setenv-customize`
- On Linux environment: rename `setenv-customize.sample.sh` in `$PLATFORM_HOME/bin` to `setenv-customize.sh` and then append this line into that file:

```
CATALINA_OPTS="$CATALINA_OPTS -Djava.security.krb5.realm=EXAMPLE.COM -Djava.security.krb5.kdc=$KDC_SERVER.example.com"
```

- On window environment: rename `setenv-customize.sample.bat` in `$PLATFORM_HOME/bin` to `setenv-customize.bat` and then append this into that file:

```
SET "CATALINA_OPTS=%CATALINA_OPTS% -Djava.security.krb5.realm=EXAMPLE.COM -Djava.security.krb5.kdc=$KDC_SERVER.example.com"
```
The `$KDC_SERVER` is either the server that installed Kerberos or the window server that domain `example.com` is running on.


##### eXo Platform Jboss
1. Add the security-domain `spnego-server` as the child of the `<security-domains>` section of the `$PLATFORM_HOME/standalone/configuration/standalone-exo.xml`:
```
<security-domain name="spnego-server" cache-type="default">
    <authentication>
        <login-module code="com.sun.security.auth.module.Krb5LoginModule" flag="required">
            <module-option name="storeKey" value="true"/>
            <module-option name="doNotPrompt" value="true"/>
            <module-option name="useKeyTab" value="true"/>
            <module-option name="keyTab" value="/etc/krb5.keytab"/>
            <module-option name="principal" value="HTTP/server.example.com@EXAMPLE.COM"/>
            <module-option name="useFirstPass" value="true"/>
            <module-option name="debug" value="true"/>
            <module-option name="isInitiator" value="false"/>
        </login-module>
    </authentication>
</security-domain>
```

2. Uncomment `SSODelegateLoginModule` in `$PLATFORM_HOME/standalone/configuration/standalone-exo.xml`
```
<login-module code="org.gatein.sso.integration.SSODelegateLoginModule" flag="required">
    <module-option name="enabled" value="#{gatein.sso.login.module.enabled}"/>
    <module-option name="delegateClassName" value="#{gatein.sso.login.module.class}"/>
    <module-option name="portalContainerName" value="portal"/>
    <module-option name="realmName" value="gatein-domain"/>
    <module-option name="password-stacking" value="useFirstPass"/>
</login-module>
```
3. Configure SSO in file `$PLATFORM_HOME/standalone/configuration/gatein/configuration.properties`:
```
# SSO
gatein.sso.enabled=true
gatein.sso.filter.spnego.enabled=true
gatein.sso.callback.enabled=false
gatein.sso.skip.jsp.redirection=false
gatein.sso.login.module.enabled=true
gatein.sso.login.module.class=org.gatein.security.sso.spnego.SPNEGOSSOLoginModule
gatein.sso.filter.login.sso.url=/@@portal.container.name@@/spnegosso
gatein.sso.filter.initiatelogin.enabled=false
gatein.sso.valve.enabled=false
gatein.sso.filter.logout.enabled=false
```

4. Start eXo Platform with command
```
./standalone.sh -Djava.security.krb5.realm=EXAMPLE.COM -Djava.security.krb5.kdc=$KDC_SERVER.example.com -Djava.security.krb5.kdc=server.example.com -b server.example.com
```
The `$KDC_SERVER` is either the server that installed Kerberos or the window server that domain `example.com` is running on.