<!-- Filename: Cache / Display title: Cache -->

O Joomla tem maneiras diferentes para guardar "coisas" em *cache*. Aqui
está um resumo para os administradores e programadores, o que, onde e
quando.

# Para Administradores

Como um administrador, o Joomla fornece-lhe a capacidade de colocar em
cache partes do seu site. Pode optar por colocar em cache páginas da Web
completas ou apenas partes dessas páginas. Este guia explica como.

Numa página da Web do *site* Joomla existem 3 coisas que podem ser
colocadas em cache:

1.  A própria página completa – a cache da Página
2.  O resultado de saída do componente Joomla para essa página da Web –
    conhecida como a *cache* da «Visualização»
3.  O resultado de saída dos módulos mostrados nessa página – conhecido
    como a *cache* do «Módulo»

Tem várias definições de cache que lhe permitem controlar o que é
colocado em *cache*:

1.  O *plug-in* do sistema "Sistema – Cache da Página"
2.  A «Configuração Global», separador do «Sistema», «Definições da
    Cache». Aqui a opção «Cache do Sistema» poderá ser definida para:
    - DESLIGADA – Cache desativada
    - LIGADA – Cache conservadora
    - LIGADA – Cache progressiva
3.  Muitos módulos dentro das próprias opções tem um separador
    «Avançado» onde pode definir a "Colocação em Cache" para "Utilizar
    global" ou "Sem *cache*»

Conforme descrito abaixo, existem também regras para colocação em cache
que são implementadas no código do Joomla, e sobre as quais não tem
controle.

Pode limpar a cache através da opção no menu do administrador Sistema /
Limpar Cache.

Em geral, pode pensar no Joomla em ter 3 níveis de cache, aumentando em
agressividade

1.  Cache conservadora
2.  Cache progressiva
3.  Cache da página

Nós iremos abordar estes três em detalhe, em baixo.

In addition, Joomla developers can use caching facilities to store the
result of database queries, for example, to increase the responsiveness
of the site, but this is outside the scope of Administrator
capabilities.

## Colocação da Página em Cache

To switch this on, go to Administrator → Extensions → Plugins. Then find
the System – Page Cache plugin, and enable it. This means that site
pages will now be cached and whenever they're requested again, the
cached page will be served, rather than it being generated by Joomla
from the information in the database. The cached page will continue to
be served until it's expired – as defined by the *Cache Time* parameter
in the Administrator → Global Configuration → System tab → Cache
Settings.

If you're reading this page as a tutorial and want to test the page
caching, it's best to set the Global Configuration cache settings to:

- Cache Handler – File
- Path to Cache Folder – leave blank
- Cache Time – 15 (the default of 15 minutes)
- Platform Specific Caching - No
- System Cache – OFF – Caching disabled

To verify that page caching is working, go to a website page that
displays an article. After you display that page you should find in the
file system a `cache/page` directory with a file in it which has a
filename like `-cache-page-.php`. (Joomla has to store separate cache
pages for separate URLs so the second string of hex digits is a hash of
the URL of the site web page, to make the filename unique to that page).

Then use the Administrator functionality to change the text of that
article, and redisplay the site web page. You should find the cached
version, not your modified text.

Changing an article (or other Joomla item) does not clear the page cache
for the web page(s) where that article is displayed. To clear the page
cache go to Administrator → System → Clear Cache. Click on the checkbox
next to the *Cache Group* called "page", and press the Delete button.
When you redisplay your web page it should now show your amended text.

If your site has a function like a shopping basket, applying page
caching will cause problems, as pages have to show what the customer has
already selected, rather than displaying a cached page which is common
to everyone. However, you can configure the System - Page Cache plugin
to exclude caching specified Menu Items or specified URLs and URL ranges
(in the Advanced tab), so that only truly static pages are cached.

## Cache Conservadora

With Conservative Caching you can cache the View output from components
and the output from those Modules which allow caching. But note that
this will work only on pages which are not cached using the Page Cache.
For those pages the whole web page is cached, and Conservative Caching
isn't even considered.

To switch on Conservative Caching:

1.  Go to Administrator → System → Global Configuration → System tab and
    within Cache Settings, set System Cache to ON – Conservative caching
2.  Go to Administrator → Extensions → Modules and select the modules
    which you'd like to be cached. If that module permits caching then
    under the Advanced tab you should be able to set Caching to

- Use Global – this module will be cached (with the Global option now
  having been set to Conservative caching)
- No caching – this module will not be cached.

(Note that the Cache Time in the Global Configuration is in minutes but
the Cache Time in the Module settings is in seconds.)

To check it's working, go to your site, **ensure that you are logged
out**, and navigate to a web page which displays an article. Check your
file system and you should find a folder `cache/com_content` containing
a cache file.

You'll also find other directories such as `cache/com_languages` (as
displaying the page involves loading the current language, and this will
be cached as well) and also directories relating to module cache, e.g.
`cache/com_modules`. These result from the use of cache which developers
have coded within the Joomla application.

If you edit and save that article then refresh the site page, you will
find that the site displays the updated text this time. This is because
whenever the edit is saved, Joomla clears the cache for that article.

However, you can demonstrate that the cache is working if you edit the
cache file in the `cache/com_content` directory using a basic text
editor. Using the editor, change one letter within the article text in
the cache file and save the file. Then when you refresh the web page you
should see the change that you made to the cache file.

How can you select which component views get cached, and under what
circumstances? Alas, you can't do this. This is determined by the Joomla
core component developers and coded in the component PHP code. The
criteria are different for each component. However, you can easily
discover what criteria are used because for each of the site components
they are coded in the site `controller.php` file. For example, at the
time of this writing (Joomla version 3.9.2) for the Contacts component
we find in `components/com_contact/controller.php`

    if (JFactory::getApplication()->getUserState('com_contact.contact.data') === null)
    {
        $cachable = true;
    }

This means that the views associated with contacts will be cachable
unless there is session data keyed by com_contact.contact.data – which
will be the case if in the user session the user has displayed a contact
form (e.g. on a page pointed to by a menu item of type Contacts → Single
Contact).

The equivalent file for articles `components/com_content/controller.php`
contains:

    $cachable = true;
    if ($user->get('id') || ($this->input->getMethod() === 'POST' && (($vName === 'category' && $this->input->get('layout') !== 'blog') || $vName === 'archive' )))
    {
        $cachable = false;
    }

The expression `$user->get('id')` is true if this is a logged-in user.
This means that articles are never cached for logged-in users. The
subsequent expressions relate to other conditions when the caching is
not performed, even if the user is not logged in.

So in this way you can discover the circumstances under which caching is
performed, but changing these is not advisable.

You can also demonstrate that modules are being cached by using the
Joomla Breadcrumbs module, ensuring it's displayed in some module
position on the web page, setting its Caching option and manually
editing the cached file in cache/mod_breadcrumbs.

## Cache Progressiva

Like Conservative Caching, Progressive Caching also caches the output
from component views and from modules. The functional difference between
the two is that with Progressive Caching **for logged-off users all
modules are always cached**. In this case, setting the *No Caching*
option for a module has no effect. If the caching storage option is to
*File*, you can find the modules cache file (the output from all modules
is stored within the same file) within the `cache/com_modules`
directory.

To switch on Progressive Caching, go to Administrator → System → Global
Configuration → System tab and within *Cache Settings* set *System
Cache* to *ON – Progressive caching*.

As regards the conditions for caching of Joomla core component views,
**there is no difference between conservative and progressive caching**.
Despite what you may read on some websites and responses to Stack
Overflow questions, it is not the case that Conservative Caching relates
to when the user is not logged on and Progressive Caching to when the
user is logged on.

## Resumo

Em baixo tem um resumo dos tipos de colocação em *cache*.

## Colocação da Página em Cache

- **Configuration**: Built-in Plugin (Administrator → Extensions →
  Plugin Manager → System - Page Cache)
- **Caches**: each whole page of your site
- **Based on**: URL
- **More info**:
  - Optional browser caching: Also caches on your visitors'
    browser/computer
  - Only caches pages for guest visitors (not for logged in visitors).
    Be careful using this plugin if you have an interactive site where
    you want to server content based on session/cookie information
    rather than on the plain URL only. Features like a shopping cart
    will not work.

### Cache de Visualização

- **Configuration**: Global Config -\> Cache
- **Caches**: each view of a component
- **Based on**: URL, view, parameters, ...
- **More info**: Component developers have to include this in their code
  to work. Mostly this is not done. The Joomla main content component
  uses this, but only for guest visitors of your site though this is not
  obligated for every component.

### Cache de Módulo

- **Configuration**: Global Config -\> Cache
- **Caches**: each module (individually customized via each module's
  Advanced Parameters)
- **Based on**: the module id, the user's view levels and the *Itemid*
  parameter in the HTTP request
- **More info**: You must disable it on some modules to avoid problems

### Cache Adicional

If you want to check out other cache systems and possibilities, you
might want to check out the third-party extensions around caching.

### Colocação de mecanismos or armazenamentos em cache

- **Configuration**: Global Config -\> Cache

Here you can choose which system you want your site to use for all
caching. Some options are: APC, Eaccelorator, File, Memcache, Redis,
XCache.

APC, for example, also caches your PHP opcode.

# Para Programadores

The class **JCache** allows a lot of different sorts and levels of
caching. The following subclasses are made specifically, but you can add
your own, or use the main one in many different ways.

Don't forget that the first level of cache encountered, will override
any deeper caching. I suppose that too many levels is also
counterproductive (*to be verified though*).

- **JCacheView** caches and returns the output of a given view (in MVC).
  A cache id is automatically generated from the URI, specific view and
  its specific method, or you can give your own.

This can automatically be done via the base controller's display
function. For example in the controller of your component:

    class DeliciousController extends JController {
        function display() {
            parent::display(true); //true asks for caching.
        }
    }

There are also some urlparams to consider. Check this <a
href="http://joomla.stackexchange.com/questions/5781/how-can-i-use-joomlas-cache-with-my-components-view/7000#7000"
class="external text" target="_blank"
rel="nofollow noreferrer noopener">"joomla stack"</a>

Also, be aware that any updates (such as hits or visit counts) will NOT
be updated (unless you add this outside this method and thus any deeper
MVC-part.)

- **JCachePage** caches and returns the body of the page.
- **JCacheCallback** caches and returns the output and results of
  functions or methods.

If you want to cache queries, this is a good class for it, as
illustrated here: [Using caching to speed up your
code](https://docs.joomla.org/Using_caching_to_speed_up_your_code "Special:MyLanguage/Using caching to speed up your code")

- **JCacheOutput** caches and returns output.

This is rather meant for caching a specific part of PHP code. It acts
like an output buffer, but cached.

## Referências

- <a
  href="http://forum.joomla.org/viewtopic.php?f=428&amp;t=326990&amp;start=0"
  class="external text" target="_blank" rel="noreferrer noopener">Melhor
  desempenho com o plug-in joomla "Cache do Sistema!" (Fórum Joomla)</a>
  (em inglês)
- <a href="https://api.joomla.org/cms-3/classes/JCache.html"
  class="external text" target="_blank" rel="noreferrer noopener">JCache
  (API do Joomla!)</a>
- <a
  href="http://joomla.stackexchange.com/questions/5781/how-can-i-use-joomlas-cache-with-my-components-view/7000#7000"
  class="external text" target="_blank"
  rel="nofollow noreferrer noopener">Como é que posso utilizar a Cache do
  Joomla com a minha visualização dos componentes? (joomla stackexchange
  beta)</a> (em inglês)