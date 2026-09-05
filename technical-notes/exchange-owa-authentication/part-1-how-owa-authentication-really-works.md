# Exchange OWA Authentication Deep Dive — Part 1

## How OWA Authentication Really Works

Exchange OWA authentication is easier to troubleshoot when the frontend, Exchange authentication layer, HttpProxy, and backend are treated as separate parts of the same request path.

A simplified flow is:

```text
Browser
  ↓
Default Web Site\owa        (frontend, HTTPS 443)
  ↓
Exchange OWA authentication
  ↓
HttpProxy / routing
  ↓
Exchange Back End\owa      (backend, HTTPS 444)
  ↓
OWA application / mailbox services
```

The normal frontend OWA baseline in this lab was:

```text
FormsAuthentication           : True
BasicAuthentication           : True
WindowsAuthentication         : False
DigestAuthentication          : False
InternalAuthenticationMethods : {Basic, Fba}
ExternalAuthenticationMethods : {Fba}
```

The important point is that Exchange OWA Forms-Based Authentication is not the same thing as IIS Forms Authentication. IIS can show Forms Authentication disabled while Exchange OWA FBA is enabled and working normally.

The frontend and backend OWA applications also use different authentication defaults. On the frontend, Basic Authentication is enabled as part of the normal FBA configuration. On the backend, Anonymous and Windows Authentication are enabled by default.

For troubleshooting, I separate three evidence layers:

```text
Frontend IIS       → W3SVC1
Exchange HttpProxy → Logging\HttpProxy\Owa
Backend IIS        → W3SVC2
```

A few practical rules from the lab:

- `WindowsAuthentication=False` on the frontend does not mean Windows Authentication is absent from the whole OWA path.
- `cs-username=-` in IIS does not by itself prove Anonymous Authentication was used.
- HTTP 401 can be either expected challenge traffic or a real failure depending on context.
- A 302 after `/owa/auth.owa` does not by itself prove the login completed successfully.
- Seamless SSO does not automatically mean Kerberos was used.

Full article with screenshots, commands, and references:

https://ceyhunkirmizitas.net/exchange-owa-authentication-deep-dive-part-1/
