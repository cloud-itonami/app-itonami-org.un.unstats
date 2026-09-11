#!/usr/bin/env nbb
(ns build-m49-crosswalk
  (:require [clojure.string :as str]))

(def fs (js/require "fs"))
(defn slurp* [p] (.toString (.readFileSync fs p)))
(defn text [html]
  (-> html
      (str/replace #"<[^>]+>" "")
      (str/replace "&nbsp;" " ")
      (str/replace "&amp;" "&")
      (str/replace "&#39;" "'")
      (str/replace "&quot;" "\"")
      str/trim))
(defn cells [row]
  (mapv (comp text second) (re-seq #"(?is)<td[^>]*>(.*?)</td>" row)))
(defn row->entry [row]
  (let [xs (cells row)
        name (get xs 8)
        m49 (get xs 9)
        iso3 (get xs 11)]
    (when (and (re-matches #"[0-9]{3}" (or m49 ""))
               (re-matches #"[A-Z]{3}" (or iso3 "")))
      {:geo/m49 m49 :country/iso3 iso3 :country/name name})))

(let [html (slurp* "raw/m49/overview-en.html")
      ;; The page contains translated copies.  The first full table is English.
      first-table (second (re-find #"(?is)<table[^>]*>(.*?)</table>" html))
      entries (->> (re-seq #"(?is)<tr[^>]*>.*?</tr>" first-table)
                   (keep row->entry)
                   distinct
                   (sort-by :geo/m49)
                   vec)]
  (when (< (count entries) 200)
    (throw (js/Error. (str "Expected at least 200 M49/ISO3 entries, got " (count entries)))))
  (.mkdirSync fs "derived" #js {:recursive true})
  (.writeFileSync fs "derived/m49-iso3.edn"
                  (str (pr-str {:crosswalk/id :un-m49-to-iso3
                                :crosswalk/source :source/un-m49-overview
                                :crosswalk/source-url "https://unstats.un.org/unsd/methodology/m49/overview/"
                                :crosswalk/retrieved-at "2026-07-10"
                                :crosswalk/entries entries}) "\n"))
  (println (str "built " (count entries) " official M49-to-ISO3 entries")))
