---
layout: post
title: Azure Mobile Apps Client API und Azure Active Directory B2C. Geht das?
author: Mephisztoe
commenting: open
---

Entwickler mobiler Lösungen auf Basis von Xamarin Forms erleben derzeit eine besondere Situation insofern sie Azure App Services einsetzen. Der Grund dafür ist u.a. der, dass es in relativ kurzer Zeit zu mehreren Rebrandings bei Microsoft gekommen ist. Leider liest Du dann nicht selten so etwas:

> Microsoft is committed to fully supporting [...], including support for the latest OS release, bug fixes,
> documentation improvements, and community PR reviews. Please note that the product team is not currently 
> investing in any new feature work for [...]. We highly appreciate community contributions to all areas of [...]."

Möchtest Du z.B. <i>einfach</i> auf Deine Azure Tabellen zugreifen, könnte es ein guter Weg sein, die Azure Mobile Apps Client API einzusetzen. Die sind nicht zu Verwechseln mit den Azure Mobile Services, da das Produkt nun ja App Services heißt. Entsprechend findest Du auf github mehrere Projekte, die oberflächlich erst einmal sehr ähnlich zu sein scheinen und wenn Du nach Informationen dazu suchst, wirst Du immer öfter den Hinweis darauf finden, die Bibliotheken seien deprecated. Mist. Denn ich weiß nicht, wie es Dir geht, aber ich persönlich mag eine integrierte Möglichkeit für Offline-Synchronisation und vor allem mag ich es, mit meinen Tabellen einfach so zu arbeiten:

{% highlight c# %}
var itemTable = await this.cloudService.GetTableAsync<Item>();
var itemList = await itemTable.ReadAllItemsAsync();
{% endhighlight %}

Bevor ich die API benutzt habe, hatte ich einen eigenen Requestprovider im Einsatz. Den brauche ich jetzt nur noch für externe Services, die keinen Client mitliefern. Das spart enorm viel ein!

Da es auf den ersten Blick nicht offensichtlich ist, welche Bibliotheken denn nun die aktuellsten sind, die nicht mehr weitergepflegt werden, hier eine Übersicht:

.NET Server SDK für Azure Mobile Apps (lies: Azure App Services)
(_Aktuelle (stabile) Version: 2.0.3_)
* Link zu [GitHub][gh-aas-server]
* Link zu [Nuget][ng-aas-server]

Azure Mobile Apps: .NET Client SDK (lies: Azure App Services)
(_Aktuelle (stabile) Version: 4.1.1_)
* Link zu [GitHub][gh-aas-client]
* Link zu [Nuget][ng-aas-client]

Okay. Nun hast Du also die aktuellsten (noch nicht ganz verwaisten) Bibliotheken am Start, um einfach mit Deinen Azure Tabellen herumhantieren zu können. Wenn das alles ist, was Dich interessiert, dann kannst Du nun wieder nach aktuellen Webcomics mit Entwickler-Bezug suchen.

Solltest Du Dich aber bereits mit dem Azure Active Directory im Allgemeinen und dem B2C Angebot im Speziellen auseinandergesetzt haben, dann könntest Du spätestens jetzt die Frage stellen, wie das denn nun mit der Authentifizierung funktioniert.

Denn:
Das .NET Client SDK für Azure App Services möchtest Du nicht für Authentifizierung nutzen sondern für den einfachen Zugriff auf Azure Tabellen. Und Azure Active Directory B2C möchtest Du nicht für die Arbeit mit Tabellen nutzen (haha, geht ja auch nicht), sondern zur Authentifizierung.

Hier also, wie Du beides zur Zusammenarbeit bewegen kannst.

Schritt 1
=========

Initialisiere die PublicClientApplication so, wie Du es gewohnt bist.
Z.B. so:

{% highlight c# %}
public static string Tenant = "DEINEAPP.onmicrosoft.com";
public static string AzureADB2CHostname = "DEINEAPP.b2clogin.com";
public static string PolicySignUpSignIn = "B2C_1_SignInSignUp";
public static string AuthorityBase = $"https://{AzureADB2CHostname}/tfp/{Tenant}/";
public static string Authority = $"{AuthorityBase}{PolicySignUpSignIn}";
...
PCA = new PublicClientApplication(ClientID, Authority);
PCA.RedirectUri = $"msal{ClientID}://auth";
PCA.ValidateAuthority = false;
{% endhighlight %}

Schritt 2
=========

Beziehe wie üblich einen Token. Silent. In etwa so:

{% highlight c# %}

IEnumerable<IAccount> accounts = await App.PCA.GetAccountsAsync();
if (accounts != null && accounts.Count() > 0)
{
	var response = Task.Run(async () => await App.PCA.AcquireTokenSilentAsync(
    	App.Scopes,
        AuthenticationHelper.GetAccountByPolicy(accounts, App.PolicySignUpSignIn),
        App.Authority,
        false));

	result = response.Result;
}

if (result != null)
	await HandleResult(result);
{% endhighlight %}

Schritt 3
=========

Tu, was immer Du musst, um mit dem Ergebnis richtig umzugehen - was immer "richtig" in Deinem Kontext heißt (bei mir z.B. lese ich aus dem Token die Object Id des AADB2C Users aus und verwende ihn als Referenz in meiner Azure-Datenbank, um eine Multi-User App zu ermöglichen). 

UND: Hier kannst Du nun das Azure .NET Client SDK initialisieren. Alles was Du brauchst, damit das funktioniert - denn auch das SDK benötigt eine korrekte Authentifizierung - ist der AccessToken aus dem Ergebnis des gerade erfolgten Logins via PublicClientApplication. Den AccessToken verpackst Du nun in einem JObject und übergibst ihn an Deinen Azure Client:

{% highlight c# %}
private async Task HandleResult(AuthenticationResult result)
{
	var zumoPayload = new JObject()
	{
		["access_token"] = result.AccessToken
	};

	await CloudService.LoginAsync(zumoPayload);
    
    ...

	// Navigate to Main
    App.Current.MainPage = new AppShell();
}


public async Task<MobileServiceUser> LoginAsync(JObject zumoPayload)
{
	return await client.LoginAsync(MobileServiceAuthenticationProvider.WindowsAzureActiveDirectory,
    	zumoPayload);
}
{% endhighlight %}

Das ist es eigentlich auch schon. Der Code, den ich hier gezeigt habe, ist nicht zum Abschreiben gedacht, sondern als Unterstützung. Im Prinzip findest Du solche und ähnliche Lösungen (sogar solche, die sich kopieren lassen), wenn Du im Internet nach dem Stichwort "ZumoPayload" suchst. Viel Erfolg!

Und vielleicht komme ich irgendwann dazu, ein funktionierendes Beispiel bei GitHub hochzuladen. Falls Du Interesse daran hast, schick' mir eine Email.

[gh-aas-server]: https://github.com/Azure/azure-mobile-apps-net-server
[ng-aas-server]: https://www.nuget.org/packages/Microsoft.Azure.Mobile.Server/
[gh-aas-client]: https://github.com/Azure/azure-mobile-apps-net-client
[ng-aas-client]: https://www.nuget.org/packages/Microsoft.Azure.Mobile.Client
