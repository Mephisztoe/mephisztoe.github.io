---
layout: post
title: FontIcons in Xamarin.Forms 4.0 Shell Tabs einsetzen
author: Mephisztoe
commenting: open
---

Vor kurzem habe ich einen spannenden Blogartikel von James Montemagno gelesen, in dem er erläutert, wie sich FontIcons gan einfach in Xamarin.Forms 4 nutzen lassen. Den Artikel findest Du [hier][jm-xf4-fonticons]. Nachdem ich die Material Design Icons in meine Projekte importiert und Dank der Website [IconFont2Code][icon-font-2-code] eine Klasse für benannte Glyphen erstellt und eingebunden habe, musste ich aber feststellen, dass ich es nicht schaffe, FontIcons für Tabs in einer Xamarin Shell anzuzeigen.

Praktisch alle Beispiele haben dazu in der AppShell.xaml unterhalb von ShellContent das Attribut ShellContent.Icon mittels FontImageSource gesetzt und hierin die IconFont referenziert. Klappt nur leider nicht.

Das hier ist die Löusng:

{% highlight xml %}

<Tab Title="About">
	<Tab.Icon>
    	<FontImageSource FontFamily="{DynamicResource MaterialFontFamily}"
         				 Glyph="{x:Static local:IconFont.InformationOutline}"
                         Size="30"
                         Color="{StaticResource IconColor}" />
	</Tab.Icon>
    <ShellContent Shell.NavBarIsVisible="False"
				  ContentTemplate="{DataTemplate views:AboutView}" />
</Tab>

{% endhighlight %}

[jm-xf4-fonticons]: https://montemagno.com/using-font-icons-in-xamarin-forms-goodbye-images-hello-fonts/
[icon-font-2-code]: https://andreinitescu.github.io/IconFont2Code/
