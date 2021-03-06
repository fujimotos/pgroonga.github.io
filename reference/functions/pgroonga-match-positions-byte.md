---
title: pgroonga_match_positions_byte function
upper_level: ../
---

# `pgroonga_match_positions_byte` function

Since 1.0.7.

## Summary

`pgroonga_match_positions_byte` function returns positions of the specified keywords in the specified text. The unit of position is byte. If you want to highlight keywords for HTML output, [`pgroonga_snippet_html` function](pgroonga-snippet-html.html) or [`pgroonga_highlight_html` function](pgroonga-highlight-html.html) will be suitable. `pgroonga_match_positions_byte` function is for advanced use.

If you want in character version, see [`pgroonga_match_positions_character`](pgroonga-match-positions-character.html) instead.

## Syntax

There are two signatures:

```text
integer[2][] pgroonga_match_positions_byte(target, ARRAY[keyword1, keyword2, ...])
integer[2][] pgroonga_match_positions_byte(target, ARRAY[keyword1, keyword2, ...], index_name)
```

The first signature is simpler than others. The first signature is enough for most cases.

The second signature is useful when you use custom normalizer.

The second signature is available since 2.2.8.

Here is the description of the first signature.

```text
integer[2][] pgroonga_match_positions_byte(target, ARRAY[keyword1, keyword2, ...])
```

`target` is a text to be searched. It's `text` type.

`keyword1`, `keyword2`, `...` are keywords to be found. They're an array of `text` type. You must specify one or more keywords.

`pgroonga_match_positions_byte` returns an array of positions.

Position consists of offset and length. Offset is the start byte from the beginning. Length is the number of bytes of matched text. Length may be different size with the length of keyword. Because keyword and matched text are normalized.

Here is the description of the second signature.

```text
integer[2][] pgroonga_match_positions_byte(target, ARRAY[keyword1, keyword2, ...], index_name)
```

`target` is a text to be searched. It's `text` type.

`keyword1`, `keyword2`, `...` are keywords to be found. They're an array of `text` type. You must specify one or more keywords.

`index_name` is an index name of the corresponding PGroonga index. It's `text` type.

`index_name` can be `NULL`.

If you aren't using `NormalizerAuto` normalizer such as `NormalizerNFKC100`, it's better that you use `index_name`. This function uses `NormalizerAuto` normalizer by default. It may cause unexpected result.

Here is an example:

```sql
CREATE TABLE memos (
  content text
);

CREATE INDEX pgroonga_content_index
          ON memos
       USING pgroonga (content)
        WITH (normalizer='NormalizerNFKC121');
```

Now, you can use `pgroonga_content_index` as `index_name`:

{% raw %}
```sql
SELECT pgroonga_match_positions_byte('Reiwa: ㋿',
                                     ARRAY['令和'],
                                     'pgroonga_content_index');
--  pgroonga_match_positions_byte 
-- -------------------------------
--  {{7,3}}
```
{% endraw %}

`pgroonga_match_positions_byte` returns an array of positions.

Position consists of offset and length. Offset is the start byte from the beginning. Length is the number of bytes of matched text. Length may be different size with the length of keyword. Because keyword and matched text are normalized.

It's available since 2.2.8.

## Usage

You need to specify at least one keyword:

{% raw %}
```sql
SELECT pgroonga_match_positions_byte('PGroonga is a PostgreSQL extension.',
                                     ARRAY['PostgreSQL']) AS match_positions_byte;
--  match_positions_byte 
-- ----------------------
--  {{14,10}}
-- (1 row)
```
{% endraw %}

You can specify multiple keywords:

{% raw %}
```sql
SELECT pgroonga_match_positions_byte('PGroonga is a PostgreSQL extension.',
                                     ARRAY['Groonga', 'PostgreSQL']) AS match_positions_byte;
--  match_positions_byte 
-- ----------------------
--  {{1,7},{14,10}}
-- (1 row)
```
{% endraw %}

You can extract keywords from query by [`pgroonga_query_extract_keywords` function](pgroonga-query-extract-keywords.html):

{% raw %}
```sql
SELECT pgroonga_match_positions_byte('PGroonga is a PostgreSQL extension.',
                                     pgroonga_query_extract_keywords('Groonga PostgreSQL -extension')) AS match_positions_byte;
--  match_positions_byte 
-- ----------------------
--  {{1,7},{14,10}}
-- (1 row)
```
{% endraw %}

Characters are normalized:

{% raw %}
```sql
SELECT pgroonga_match_positions_byte('PGroonga + pglogical = replicatable!',
                                     ARRAY['Pg']) AS match_positions_byte;
--  match_positions_byte 
-- ----------------------
--  {{0,2},{11,2}}
-- (1 row)
```
{% endraw %}

Multibyte characters are also supported:

{% raw %}
```sql
SELECT pgroonga_match_positions_byte('10㌖先にある100ｷﾛグラムの米',
                                     ARRAY['キロ']) AS match_positions_byte;
--  match_positions_byte 
-- ----------------------
--  {{2,3},{20,6}}
-- (1 row)
```
{% endraw %}

Custom normalizer can be used by specifying a PGroonga index name:

{% raw %}
```sql
CREATE TABLE memos (
  content text
);

CREATE INDEX pgroonga_content_index
          ON memos
       USING pgroonga (content)
        WITH (normalizer='NormalizerNFKC121');

SELECT pgroonga_match_positions_byte('Reiwa: ㋿',
                                     ARRAY['令和'],
                                     'pgroonga_content_index');
--  pgroonga_match_positions_byte 
-- -------------------------------
--  {{7,3}}
-- (1 row)
```
{% endraw %}

## See also

  * [`pgroonga_match_positions_character` function][match-positions-character]

  * [`pgroonga_snippet_html` function][query-snippet-html]

  * [`pgroonga_highlight_html` function][query-highlight-html]

  * [`pgroonga_query_extract_keywords` function][query-extract-keywords]

[match-positions-character]:pgroonga-match-positions-character.html
[query-snippet-html]:pgroonga-query-snippet-html.html
[query-highlight-html]:pgroonga-query-highlight-html.html
[query-extract-keywords]:pgroonga-query-extract-keywords.html

