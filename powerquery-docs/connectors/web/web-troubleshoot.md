---
title: Troubleshooting the Power Query Web connector
description: Provides troubleshooting tips for errors that might occur when using the Power Query Web connector to connect to a web site.
author: bezhan-msft
ms.topic: conceptual
ms.date: 2/1/2023
ms.author: bezhan
---

# Troubleshooting the Web connector

## What's the difference between Web.Contents, Web.BrowserContents, and Web.Page?

[Web.Contents](/powerquery-m/web-contents)

* `Web.Contents` is used for retrieving web content that doesn't need to be accessed through a browser, such as CSV files, JSON API results, and so on.
* It supports the widest variety of authentication options.
* It can be used in cloud environments, such as Power Query Online, without a gateway.

[Web.Page](/powerquery-m/web-page)

* `Web.Page` is a legacy function for retrieving web content that needs to be accessed through a browser, such as HTML pages.
* It's built on Internet Explorer. Because of this requirement, it's being replaced in the UI with `Web.BrowserContents`. However, `Web.Page` will continue to be available at the engine level for backward compatibility.
* A gateway is required to use it in cloud environments, such as Power Query Online.

[Web.BrowserContents](/powerquery-m/web-browsercontents)

* `Web.BrowserContents` is a new function for retrieving web content that needs to be accessed through a browser, such as HTML pages.
* In the UI, `Web.BrowserContents` is replacing `Web.Page`, because `Web.Page` is based on Internet Explorer.
* `Web.BrowserContents` was initially built on Chromium, but [it's being migrated to Microsoft Edge's WebView2 control](#enabling-the-edge-webview2-version-of-webbrowsercontents).
* A gateway is required to use it in cloud environments, such as Power Query Online.

The following table summarizes the differences.

| | [Web.Contents](/powerquery-m/web-contents) | [Web.Page](/powerquery-m/web-page) | [Web.BrowserContents](/powerquery-m/web-browsercontents) |
|----|----|----|----|
| **Non-browser content (.txt/.csv files, JSON, and so on)** | x | | |
| **Browser content (HTML)** | | x | x |
| **Authentication Types Supported** | Anonymous<br/>Windows<br/>Basic<br/>Web API<br/>Organizational Account | Anonymous<br/>Windows (current user's credentials only)<br/>Web API | Anonymous<br/>Windows ([preview feature](#enabling-the-edge-webview2-version-of-webbrowsercontents))<br/>Basic ([preview feature](#enabling-the-edge-webview2-version-of-webbrowsercontents))<br/>Web API ([preview feature](#enabling-the-edge-webview2-version-of-webbrowsercontents)) |
| **Requires a gateway in cloud hosts** | N | Y | Y |
| **Currently generated by** | All hosts | Excel and Power Query Online | Power BI Desktop |
| **Built on** | .NET | Internet Explorer | Chromium, but [moving to Microsoft Edge's WebView2 control](#enabling-the-edge-webview2-version-of-webbrowsercontents) |

## Handling dynamic web pages

Web pages that load their content dynamically might require special handling. If you notice sporadic errors in your web queries, it's possible that you're trying to access a dynamic web page. One common example of this type of error is:

1. You refresh a query that connects to the site.
2. You see an error (for example, "the column 'Foo' of the table wasn't found").
3. You refresh the query again.
4. No error occurs.

These kinds of issues are usually due to timing. Pages that load their content dynamically can sometimes be inconsistent since the content can change after the browser considers loading complete. Sometimes the web connector downloads the HTML after all the dynamic content has loaded. Other times the changes are still in progress when it downloads the HTML, leading to sporadic errors.

The solution is to use the `WaitFor` option of [Web.BrowserContents](/powerquery-m/web-browsercontents), which indicates either a selector or a length of time that should be waited for before downloading the HTML.

How can you tell if a page is dynamic? Usually it's pretty simple. Open the page in a browser and watch it load. If the content shows up right away, it's a regular HTML page. If it appears dynamically or changes over time, it's a dynamic page.

## Using a gateway with the Web connector

Both [Web.BrowserContents](/powerquery-m/web-browsercontents) and [Web.Page](/powerquery-m/web-page) require the use of an on-premises data gateway when published to a cloud service, such as Power BI datasets or dataflows, or Power Apps dataflows. (Currently, Dynamics 365 Customer Insights doesn't support the use of a gateway.)

If you're using [Web.Page](/powerquery-m/web-page) and receive a `Please specify how to connect` error, ensure that you have Internet Explorer 10 or later installed on the machine that hosts your on-premises data gateway. 

## Enabling the Edge WebView2 version of Web.BrowserContents

[Web.BrowserContents](/powerquery-m/web-browsercontents) was initially built on Chromium, but it's being migrated to Edge's WebView2 control.

To enable the updated Edge-based version of `Web.BrowserContents` in Power BI Desktop, enable the "Web page connector infrastructure update" preview feature.

To enable the updated Edge-based version of Web.BrowserContents on a gateway machine, set the following environment variable:

`set PQ_WebView2Connector=true`

## Using Web.Page instead of Web.BrowserContents

In cases where you need to use `Web.Page` instead of `Web.BrowserContents`, you can still manually use `Web.Page`.

In Power BI Desktop, you can use the older `Web.Page` function by clearing the **New web table inference** preview feature:

1. Under the **File** tab, select **Options and settings** > **Options**.
2. In the **Global** section, select **Preview features**.
3. Clear the **New web table inference** preview feature, and then select **OK**.
4. Restart Power BI Desktop.

    >[!NOTE]
    >Currently, you can't turn off the use of `Web.BrowserContents` in Power BI Desktop optimized for Power BI Report Server.

You can also get a copy of a `Web.Page` query from Excel. To copy the code from Excel:

1. Select **From Web** from the **Data** tab.
2. Enter the address in the **From Web** dialog box, and then select **OK**.
3. In **Navigator**, choose the data you want to load, and then select **Transform Data**.
4. In the **Home** tab of Power Query, select **Advanced Editor**.
5. In the **Advanced Editor**, copy the M formula.
6. In the app that uses `Web.BrowserContents`, select the **Blank Query** connector.
7. If you're copying to Power BI Desktop:
    1. In the **Home** tab, select **Advanced Editor**.
    2. Paste the copied `Web.Page` query in the editor, and then select **Done**.
8. If you're copying to Power Query Online:
    1. In the **Blank Query**, paste the copied `Web.Page` query in the blank query.
    2. Select an on-premises data gateway to use.
    3. Select **Next**.

You can also manually enter the following code into a blank query. Ensure that you enter the address of the web page you want to load.

```powerqury-m
let
  Source = Web.Page(Web.Contents("<your address here>")),
  Navigation = Source{0}[Data]
in
  Navigation
```

## Capturing web requests and certificate revocation

We've strengthened the security of web connections to protect your data. However, this means that certain scenarios, like capturing web requests with Fiddler, will no longer work by default. To enable those scenarios:

1. Open Power BI Desktop.
2. Under the **File** tab, select **Options and settings** > **Options**.
3. In **Options**, under **Global** > **Security**, uncheck **Enable certificate revocation check**.

   ![Screenshot of the Enable certificate revocation check box selected.](web-certificate-revocation.png)

4. Select **OK**.
5. Restart Power BI Desktop.

>[!IMPORTANT]
>Be aware that unchecking **Enable certificate revocation check** will make web connections less secure.

To set this scenario in Group Policy, use the "DisableCertificateRevocationCheck" key under the registry path "Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Microsoft Power BI Desktop". Setting "DisableCertificateRevocationCheck" to 0 will always enable the check (stopping Fiddler and similar software from working) and setting "DisableCertificateRevocationCheck" to 1 will always disable the check (enabling Fiddler and similar software).

## Changing the authentication method

In some cases, you may need to change the authentication method you use to access a particular site. If this change is necessary, go to [Change the authentication method](../../connectorauthentication.md#change-the-authentication-method).

## Authenticating to arbitrary services

Some services support the ability for the Web connector to authenticate with OAuth/AAD authentication out of the box. However, this won't work in most cases.

When attempting to authenticate, if you see the following error:

“We were unable to connect because this credential type isn’t supported for this resource. Please choose another credential type.”

   ![Error from connecting to an endpoint that doesn't support OAuth with the web connector.](./credential-type-not-supported.png)

Contact the service owner. They'll either need to change the authentication configuration or build a custom connector.

## Web connector uses HTTP 1.1 to communicate

The Power Query Web connector communicates with a data source using HTTP 1.1. If your data source is expecting to communicate using HTTP 1.0, you might receive an error, such as `500 Internal Server Error`.

It's not possible to switch Power Query to use HTTP 1.0. Power Query always sends an `Expect:100-continue` when there's a body to avoid passing a possibly large payload when the initial call itself might fail (for example, due to a lack of permissions). Currently, this behavior can't be changed.

## Connecting to Microsoft Graph

Power Query currently doesn't support connecting to [Microsoft Graph](/graph/overview) REST [APIs](https://graph.microsoft.com). More information: [Lack of Support for Microsoft Graph in Power Query](../../connecting-to-graph.md)

### See also

* [Power Query Web connector](web.md)
* [Get webpage data by providing examples](web-by-example.md)