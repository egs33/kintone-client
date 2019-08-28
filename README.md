# kintone-clj

A [kintone](https://www.kintone.com) SDK for Clojure and ClojureScript.

## Overview

The SDK provides an easy way to use kintone API from Clojure or ClojureScript.

Every API run asynchronously to use [clj-http](https://github.com/dakrone/clj-http) for Clojure
, [cljs-ajax](https://github.com/JulianBirch/cljs-ajax) for ClojureScript and
[core.async](https://github.com/clojure/core.async).
They return a channel of core.async.

- `kintone.authentication` : Make Auth object.
- `kintone.connection` : Make connection object.
- `kintone.types` : Type definitions such as response object.
- `kintone.record` : kintone REST Record API.

## Usage

Follow bellow the steps.

1. Make `Auth` object
1. Make `Connection` object
1. Call API with `Connection` object

### Make `Auth` object

`Auth` object is used in order to make `Connection` object. 

This step is not necessary in case that you run JavaScript(ClojureScript) 
on kintone as customise script for kintone app or portal.

```clojure
(require '[kintone.authentication :as auth])

;; API token
(auth/new-auth {:api-token "xyz..."})

;; Basic authentication and passwrod authentication
(auth/new-auth {:basic {:username "basic-username" :password "basic-password"}
                :password {:username "login-name" :password "login-password"}})

;; Basic authentication, passwrod authentication and API token
;; In this case, the API token is going to be ignored, 
;; and the basic authentication and the password authentication is going to be used.
(auth/new-auth {:basic {:username "basic-username" :password "basic-password"}
                :password {:username "login-name" :password "login-password"}
                :api-token "xyz..."})
```

### Make `Connection` object

```clojure
(require '[kintone.authentication :as auth])
(require '[kintone.connection :as conn])

;; Auth and domain
(conn/new-connection {:auth (auth/new-auth {:api-token "xyz.."})
                      :domain "sample.kintone.com"})

;; To guest space
(conn/new-connection {:auth (auth/new-auth {:api-token "xyz.."})
                      :domain "sample.kintone.com"
                      :guest-space-id 1})

;; It is noly accepted on ClojureScript that there is no auth.
(conn/new-connection {:domain "sample.cybozu.com"})
```

### Call API

You should use `Connection` object as the first argument on every API call.

```clojure
(require '[kintone.authentication :as auth])
(require '[kintone.connection :as conn])
(require '[kintone.record :as record])

(def conn (conn/new-connection {:auth (auth/new-auth {:api-token "xyz.."})
                                :domain "sample.kintone.com"}))

;; Clojure
(require '[clojure.core.async :refer [<!!]])

(let [app 1111
      id 1
      ;; Block the thread and get response
      res (<!! (record/get-record conn app id))]
  ;; Every API response is kintone.types/KintoneResonse
  (if (:err res)
    (log/error "Something bad happen")
    (:res res)))
;; success => {:record {:$id {:type "__ID__", :value "1"} ...}
;; fail => {:status 404
;;          :status-text "Not Found"
;;          :failure :erro
;;          :response {:code "GAIA_RE01" ...}}


;; ClojureScript
(require '[cljs.core.async :refer [<!] :refer-macros [go]])

;; Call in go block to handle response
(go
  (let [app 1111
        id 1
        res (<! (record/get-record conn app id))]
    (if (:err res)
      (:err res)
      (:res res))))
```

For more information, See API documents.

## License

Copyright © 2019 TOYOKUMO,Inc.

This program and the accompanying materials are made available under the
terms of the Eclipse Public License 2.0 which is available at
http://www.eclipse.org/legal/epl-2.0.

This Source Code may also be made available under the following Secondary
Licenses when the conditions for such availability set forth in the Eclipse
Public License, v. 2.0 are satisfied: GNU General Public License as published by
the Free Software Foundation, either version 2 of the License, or (at your
option) any later version, with the GNU Classpath Exception which is available
at https://www.gnu.org/software/classpath/license.html.
