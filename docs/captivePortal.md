
<h1>Social Login</h1>

La funzionalità di Social login è garantita generando un token temporaneo dopo che l'utente ha effettuato il login tramite Social. Questo tipo di implementazione permette di implementare il social login con un qualsiasi captive portal che supporta il protocollo RADIUS perchè è totalmente trasparente ad esso.

Installazione di django-allauth:

pip install django-allauth

Assicurarsi che il file settings.py e come l'esempio seguente (Sarà spiegato utilizzando facebook come esempio)

INSTALLED_APPS = [
    # ... other apps ..
    # apps needed for social login
    'rest_framework.authtoken',
    'django.contrib.sites',
    'allauth',
    'allauth.account',
    'allauth.socialaccount',
    # showing facebook as an example
    # to configure social login with other social networks
    # refer to the django-allauth documentation
    'allauth.socialaccount.providers.facebook',
]

SITE_ID = 1

Al file urls.py va aggiunto anche il la seguente linea tra gli url

urlpatterns = [
    # .. other urls ...
    url(r'^accounts/', include('allauth.urls')),
]

Facebook supporta sia OAuth2 che il Facebook Connect Javascript SDK.

Un vantaggio dell'SDK di Javascript potrebbe essere un'esperienza utente più ottimizzata poiché non si esce dal sito. Inoltre, non è necessario preoccuparsi di adattare la finestra di dialogo di accesso a seconda che si stia utilizzando o meno un dispositivo mobile. Tuttavia, fare affidamento su Javascript potrebbe non essere sempre la soluzione migliore.

The following Facebook settings are available:

SOCIALACCOUNT_PROVIDERS = {
    'facebook': {
        'METHOD': 'oauth2',
        'SDK_URL': '//connect.facebook.net/{locale}/sdk.js',
        'SCOPE': ['email', 'public_profile', 'user_friends'],
        'AUTH_PARAMS': {'auth_type': 'reauthenticate'},
        'INIT_PARAMS': {'cookie': True},
        'FIELDS': [
            'id',
            'email',
            'name',
            'first_name',
            'last_name',
            'verified',
            'locale',
            'timezone',
            'link',
            'gender',
            'updated_time',
        ],
        'EXCHANGE_TOKEN': True,
        'LOCALE_FUNC': 'path.to.callable',
        'VERIFIED_EMAIL': False,
        'VERSION': 'v2.12',
    }
}

METHOD:
    js_sdk or oauth2. Di default è oauth2.
SDK_URL:
    se necessario, è possibile utilizzare un SDK_URL per sovrascrivere il JavaScript Facebook URL SDK (//connect.facebook.net/{locale}/sdk.js) Questo può essere per esempio necessario quando si decide di un plugin di chat clienti.Se SDK_URL contiene una stringa chiamata {locale} vuol dire che la funzione LOCALE_FUNC sarà usata per generare SDK_URL.
SCOPE:
    Per impostazione predefinita, l'ambito e-mail è richiesto a seconda che SOCIALACCOUNT_QUERY_EMAIL sia abilitato o meno. Le app che utilizzano autorizzazioni oltre a e-mail, public_profile e user_friends richiedono la revisione da parte di Facebook. Vedi Autorizzazioni con Facebook Login per ulteriori informazioni.
AUTH_PARAMS:
    Utilizzare AUTH_PARAMS per passare altri parametri alla chiamata SDK JS FB.login.
FIELDS:
    I campi da recuperare dall'API Graph / me /? fields = endpoint. Ad esempio, potresti aggiungere il campo "amici" per catturare gli amici dell'utente che hanno effettuato l'accesso anche alla tua app tramite Facebook (richiede l'ambito "user_friends").
EXCHANGE_TOKEN:
    L'SDK JS restituisce un token di breve durata adatto all'uso lato client. Impostare EXCHANGE_TOKEN = True per effettuare una richiesta lato server per l'aggiornamento a un token di lunga durata prima di archiviare nel record SocialToken.
LOCALE_FUNC:

    The locale for the JS SDK is chosen based on the current active language of the request, taking a best guess. This can be customized using the LOCALE_FUNC setting, which takes either a callable or a path to a callable. This callable must take exactly one argument, the request, and return a valid Facebook locale as a string, e.g. US English:

    SOCIALACCOUNT_PROVIDERS = {
        'facebook': {
            'LOCALE_FUNC': lambda request: 'en_US'
        }
    }

VERIFIED_EMAIL:
    It is not clear from the Facebook documentation whether or not the fact that the account is verified implies that the e-mail address is verified as well. For example, verification could also be done by phone or credit card. To be on the safe side, the default is to treat e-mail addresses from Facebook as unverified. But, if you feel that is too paranoid, then use this setting to mark them as verified. Due to lack of an official statement from the side of Facebook, attempts have been made to reverse engineer the meaning of the verified flag. Do know that by setting this to True you may be introducing a security risk.
VERSION:
    The Facebook Graph API version to use. The default is v2.12.
App registration (get your key and secret here)
    A key and secret key can be obtained by creating an app. After registration you will need to make it available to the public. In order to do that your app first has to be reviewed by Facebook.
Development callback URL
    Leave your App Domains empty and put http://localhost:8000 in the section labeled Website with Facebook Login. Note that you’ll need to add your site’s actual domain to this section once it goes live.

Captive page button example

Following the previous example configuration with facebook, in your captive page you will need an HTML button similar to the ones in the following examples.

<a href="https://openwisp2.mywifiproject.com/accounts/facebook/login/?next=%2Ffreeradius%2Fsocial-login%2F%3Fcp%3Dhttps%3A%2F%2Fcaptivepage.mywifiproject.com%2F%26last%3D"
   class="button">Log in with Facebook
</a>

Substitute openwisp2.mywifiproject.com and captivepage.mywifiproject.com with the hostname of your django-freeradius instance and your captive page respectively.
