ASP.NET Content-Security-Policy script-nonce
====================

Add Content-Security-Policy script-nonce to razor views

http://www.w3.org/TR/CSP2/#directive-script-src


```csharp
  // Startup.cs
	// https://vcsjones.com/2014/12/17/content-security-policy-nonces-in-asp-net-and-owin/	
	
	using Microsoft.Owin;
	using Owin;
	using System;
	using System.Security.Cryptography;
	using System.Web;
	using System.Web.Mvc;
		
    public partial class Startup
    {
        public void Configuration(IAppBuilder app)
        {
            
            app.Use((context, next) =>
            {
                var rng = new RNGCryptoServiceProvider();
                var nonceBytes = new byte[32];
                rng.GetBytes(nonceBytes);
                var nonce = Convert.ToBase64String(nonceBytes);
                context.Set("ScriptNonce", nonce);
                context.Response.Headers.Add("Content-Security-Policy",
                    new[] { string.Format("script-src 'self' 'nonce-{0}' 'unsafe-inline' 'strict-dynamic' https: http: 'unsafe-eval';object-src 'none';base-uri 'self';", nonce) });

                HttpContext.Current.Items["ScriptNonce"] = nonce;

                return next();
            });
        }
    }

    public static class NonceHelper
    {
        public static IHtmlString ScriptNonce(this HtmlHelper helper)
        {
            var owinContext = helper.ViewContext.HttpContext.GetOwinContext();
            return new HtmlString(owinContext.Get<string>("ScriptNonce"));
        }
    }
```


```csharp
<script nonce="@Html.ScriptNonce()"></script>
```

```csharp
@ContentSesurityPolicy.Render("~/bundle/example.js")
```
