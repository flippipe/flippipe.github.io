---
title: "Firefox Policies Json"
date: 2022-05-25T21:53:14+01:00
summary: "Customizing Firefox Using policies.json"
description: "Customizing Firefox Using policies.json"
tags: ["firefox"]
draft: false
---

Firefox allow to create a configuration file to impose policies to the users.

This [article](https://support.mozilla.org/en-US/kb/customizing-firefox-using-policiesjson) gives an overview about `policies.json`, but the [full documentation](https://github.com/mozilla/policy-templates/blob/master/README.md) it is in Github.


In short, create the file with your desired policy and place it in:

``` Text 
/etc/firefox/
├── policies
│   └── policies.json
├── pref
│   └── apturl.js
└── syspref.js
```

Then, in url bar just type `about:policies` to show all configurations applied.

This is a nice way to keep all my firefox installations similar.

An example, of a `policies.json` file:


``` json
{
  "policies": {
    "SSLVersionMin": "tls1.2",
    "WindowsSSO": false,
    "ShowHomeButton": false,
    "SearchSuggestEnabled": false,
    "PasswordManagerEnabled": false,
    "OfferToSaveLoginsDefault": false,
    "OfferToSaveLogins": false,
    "ExtensionUpdate": true,
    "DisableTelemetry": false,
    "DisablePocket": true,
    "DisableFirefoxAccounts": true,
    "CaptivePortal": true,
    "DontCheckDefaultBrowser": true,
    "WebsiteFilter": {
      "Block": [
        "https://www.bing.com"
      ],
      "Exceptions": [
        "http://example.org/*"
      ]
    },
    "UserMessaging": {
      "WhatsNew": false,
      "ExtensionRecommendations": false,
      "FeatureRecommendations": false,
      "UrlbarInterventions": false,
      "SkipOnboarding": false,
      "MoreFromMozilla": false
    },
    "SanitizeOnShutdown": {
      "Cache": true,
      "Cookies": true,
      "Downloads": true,
      "FormData": true,
      "History": false,
      "Sessions": true,
      "SiteSettings": false,
      "OfflineApps": true,
      "Locked": true
    },
    "RequestedLocales": [
      "en-GB"
    ],
    "PopupBlocking": {
      "Allow": [
        "http://example.org/",
        "http://example.edu/"
      ],
      "Default": true,
      "Locked": false
    },
    "Permissions": {
      "Camera": {
        "Allow": [
          "https://example.org",
          "https://example.org:1234"
        ],
        "Block": [
          "https://example.edu"
        ],
        "BlockNewRequests": false,
        "Locked": false
      },
      "Microphone": {
        "Allow": [
          "https://example.org"
        ],
        "Block": [
          "https://example.edu"
        ],
        "BlockNewRequests": false,
        "Locked": false
      },
      "Location": {
        "Allow": [
          "https://example.org"
        ],
        "Block": [
          "https://example.edu"
        ],
        "BlockNewRequests": true,
        "Locked": true
      },
      "Notifications": {
        "Allow": [
          "https://example.org"
        ],
        "Block": [
          "https://example.edu"
        ],
        "BlockNewRequests": true,
        "Locked": true
      },
      "Autoplay": {
        "Allow": [
          "https://example.org"
        ],
        "Block": [
          "https://example.edu"
        ],
        "Default": "block-audio-video",
        "Locked": true
      }
    },
    "Handlers": {
      "ver mais tarde com calma": "https://github.com/mozilla/policy-templates/blob/master/README.md#handlers=",
      "mimeTypes": {
        "application/msword": {
          "action": "useSystemDefault",
          "ask": false
        }
      },
      "schemes": {
        "mailto": {
          "action": "useSystemDefault",
          "ask": true
        }
      },
      "extensions": {
        "teste": {
          "action": "useHelperApp",
          "ask": true,
          "handlers": [
            {
              "name": "Adobe Acrobat",
              "path": "/usr/bin/acroread"
            }
          ]
        }
      }
    },
    "ExtensionSettings": {
      "*": {
        "blocked_install_message": "Call yourself to allow",
        "install_sources": [
          "about:addons",
          "https://addons.mozilla.org/"
        ],
        "installation_mode": "allowed",
        "allowed_types": [
          "extension",
          "theme",
          "dictionary",
          "locale"
        ]
      },
      "addons-search-detection@mozilla.com": {
        "installation_mode": "blocked",
        "install_url": ""
      },
      "bing@search.mozilla.org": {
        "installation_mode": "blocked",
        "install_url": ""
      },
      "ebay@search.mozilla.org": {
        "installation_mode": "blocked"
      },
      "google@search.mozilla.org": {
        "installation_mode": "blocked"
      },
      "{74145f27-f039-47ce-a470-a662b129930a}": {
        "installation_mode": "force_installed",
        "install_url": "https://addons.mozilla.org/firefox/downloads/latest/clearurls/latest.xpi"
      },
      "addon@darkreader.org": {
        "installation_mode": "force_installed",
        "install_url": "https://addons.mozilla.org/firefox/downloads/latest/darkreader/latest.xpi"
      },
      "@contain-facebook": {
        "installation_mode": "force_installed",
        "install_url": "https://addons.mozilla.org/firefox/downloads/latest/facebook-container/latest.xpi"
      },
      "@testpilot-containers": {
        "installation_mode": "force_installed",
        "install_url": "https://addons.mozilla.org/firefox/downloads/latest/multi-account-containers/latest.xpi"
      },
      "jid1-MnnxcxisBPnSXQ@jetpack": {
        "installation_mode": "force_installed",
        "install_url": "https://addons.mozilla.org/firefox/downloads/latest/privacy-badger17/latest.xpi"
      },
      "{c607c8df-14a7-4f28-894f-29e8722976af}": {
        "installation_mode": "force_installed",
        "install_url": "https://addons.mozilla.org/firefox/downloads/latest/temporary-containers/latest.xpi"
      },
      "uBlock0@raymondhill.net": {
        "installation_mode": "force_installed",
        "install_url": "https://addons.mozilla.org/firefox/downloads/latest/ublock-origin/latest.xpi"
      },
      "pt-PT@dictionaries.addons.mozilla.org": {
        "installation_mode": "force_installed",
        "install_url": "https://addons.mozilla.org/en-US/firefox/addon/portugu%C3%AAs-portugal-language/"
      }
    },
    "EnableTrackingProtection": {
      "Value": true,
      "Locked": false,
      "Cryptomining": true,
      "Fingerprinting": true,
      "Exceptions": [
        "https://example.com"
      ]
    },
    "DNSOverHTTPS": {
      "Enabled": false,
      "ProviderURL": "URL_TO_ALTERNATE_PROVIDER",
      "Locked": false,
      "ExcludedDomains": [
        "example.com"
      ]
    },
    "DisableSecurityBypass": {
      "InvalidCertificate": false,
      "SafeBrowsing": false
    },
    "DisabledCiphers": {
      "TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256": false,
      "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA38": false,
      "TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384": false,
      "TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256": false,
      "TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256": false,
      "TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384": false,
      "TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256": false,
      "TLS_DHE_RSA_WITH_AES_128_CBC_SHA": true,
      "TLS_DHE_RSA_WITH_AES_256_CBC_SHA": true,
      "TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA": true,
      "TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA": true,
      "TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA": true,
      "TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA": true,
      "TLS_RSA_WITH_3DES_EDE_CBC_SHA": true,
      "TLS_RSA_WITH_AES_128_CBC_SHA": true,
      "TLS_RSA_WITH_AES_128_GCM_SHA256": true,
      "TLS_RSA_WITH_AES_256_CBC_SHA": true,
      "TLS_RSA_WITH_AES_256_GCM_SHA384": true
    },
    "Cookies": {
      "Allow": [
        "https://example.org/"
      ],
      "AllowSession": [
        "http://example.edu/"
      ],
      "Block": [
        "http://example.edu/"
      ],
      "ExpireAtSessionEnd": true,
      "Locked": false,
      "Behavior": "limit-foreign",
      "BehaviorPrivateBrowsing": "reject-tracker-and-partition-foreign"
    }
  }
}

```