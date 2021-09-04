`hass` enables you to control Home Assistant entities from Emacs.

# Installation

## straight.el

``` emacs-lisp
(straight-use-package
  '(hass
    :type git
    :host github
    :repo "purplg/hass"))
```

## Doom Emacs

Place in your `packages.el` to pull the repository.

``` emacs-lisp
(package! hass
  :recipe
  '(:host github
    :repo "purplg/hass"))
```

Then load the package in your main config file.

``` emacs-lisp
(use-package! hass)
```

# Configuration

Both `hass-url` and `hass-apikey` must be set to use this package

``` emacs-lisp
(setq hass-url "https://192.168.1.10:8123"
      hass-apikey "APIKEY-GOES-IN-HERE")
```

Alternatively, you can store a lambda function inside `hass-apikey`.
This will be executed on every query.

``` emacs-lisp
(setq hass-apikey (lambda () (auth-source-pass-get 'secret "home/hass/emacs-apikey")))
```

The `hass-entity` variable is used when the `hass-query-all-entities`
function is called. It should contain a list of strings of entity ID's
for each entity you want included.

## Getting an API Key

Ensure that your Home Assistant instance is configure to support API
calls by following the instructions
[here](https://www.home-assistant.io/integrations/api/).

Retrieve your API key a.k.a. *Long-Lived Access Token* by logging into
your Home Assistant instance and going to your profile by selecting your
username in the lower-left corner or going to this URL:
`http://HOME-ASSISTANT-URL:8123/profile`. You can generate an API token
at the very bottom of this page.

# Usage

To call a service on Home Assistant, use the `hass-call-service`
function which has two required arguments: `entity-id` and `service`.
The domain of the service is detected from first part of the
`entity-id`. For example, `switch.bedroom_light` is in the `switch`
domain. Currently, the only valid services available are `turn-on`,
`turn-off`, and `toggle` and the only supported domains are `switch` and
`input_boolean`.

``` emacs-lisp
(hass-call-service "switch.bedroom_light" "toggle")
```

If you call `hass-call-service` interactively, it will prompt you for an
entity ID and then the respective service you want to call.

## Hooks

The most useful hook is a function list named
`hass-entity-state-updated-functions`. Functions in this list are passed
a single argument `entity-id` which is the entity id of the entity whose
state has changed since it was last updated.

This example will display the state of an entity when it changes:

``` emacs-lisp
(add-hook 'hass-entity-state-updated-functions
  (lambda (entity-id)
    (message "The entity %s state has changed to %s." entity-id (hass-state-of entity-id))))
```

The other two hooks available are `hass-entity-state-updated-hook` and
`hass-service-called-hook`. `hass-entity-state-updated-hook` is called
when the state of an entity is updated, regardless of if it changed or
not. `hass-service-called-hook` is called when a service is called.

``` emacs-lisp
(add-hook 'hass-service-called-hook (lambda () (message "A service was called.")))
(add-hook 'hass-entity-state-updated-hook (lambda () (message "An entitys' state was updated.")))
```

## Auto-query

Auto-querying is a recurring query to the Home Assistant instance to get
the current state of some entities. The list of entity IDs that will be
queried are found in the variable `hass-entities`.

``` emacs-lisp
(setq hass-entities '("switch.bedroom_light" "switch.bedroom_fan"))
```

The frequency of the query can be adjusted by setting
`hass-auto-query-frequency` to the number of seconds you'd like.
Defaults to 60.

Auto-querying is most useful with the function hook
`hass-entity-state-updated-functions` explained in the [Hooks](#Hooks) section.