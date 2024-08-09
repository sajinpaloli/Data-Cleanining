<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Data Cleaning - Movies Dataset</title>
</head>
<body>

<h1>Data Cleaning - Movies Dataset</h1>

<h2>Overview</h2>
<p>This project focuses on cleaning and preprocessing the <code>movies_metadata.csv</code> dataset to prepare it for analysis. The dataset contains various columns, including some that are nested or stringified JSON objects, which require special handling. The steps involved in this cleaning process include dropping irrelevant columns, flattening nested data, converting data types, handling missing values, and more.</p>

<h2>Dataset</h2>
<p>The dataset used is <code>movies_metadata.csv</code>, which contains metadata for various movies, including titles, genres, production companies, and more.</p>

<h2>Steps to Clean the Dataset</h2>

<h3>1. Loading the CSV File</h3>
<ul>
    <li><strong>Objective:</strong> Load and inspect the dataset to identify columns with nested or stringified JSON data.</li>
    <li><strong>Code Snippet:</strong></li>
</ul>
<pre><code>import pandas as pd
pd.options.display.max_columns = 30

df = pd.read_csv("movies_metadata.csv", low_memory=False)
df.info()
</code></pre>

<h3>2. Dropping Irrelevant Columns</h3>
<ul>
    <li><strong>Objective:</strong> Drop columns that are not necessary for analysis, such as <code>adult</code>, <code>imdb_id</code>, <code>original_title</code>, <code>video</code>, and <code>homepage</code>.</li>
    <li><strong>Code Snippet:</strong></li>
</ul>
<pre><code>df.drop(columns=["adult", "imdb_id", "original_title", "video", "homepage"], inplace=True)
</code></pre>

<h3>3. Handling Stringified JSON Columns</h3>
<ul>
    <li><strong>Objective:</strong> Convert stringified JSON columns into proper JSON objects and handle them accordingly.</li>
    <li><strong>Code Snippet:</strong></li>
</ul>
<pre><code>import ast

json_cols = ["belongs_to_collection", "genres", "production_countries", "production_companies", "spoken_languages"]
df[json_cols] = df[json_cols].applymap(lambda x: ast.literal_eval(x) if isinstance(x, str) else x)
</code></pre>

<h3>4. Flattening Nested Columns</h3>
<ul>
    <li><strong>Objective:</strong> Extract and flatten nested data from columns like <code>belongs_to_collection</code>, <code>genres</code>, etc.</li>
    <li><strong>Code Snippet:</strong></li>
</ul>
<pre><code>df["belongs_to_collection"] = df["belongs_to_collection"].apply(lambda x: x['name'] if isinstance(x, dict) else None)
df["genres"] = df["genres"].apply(lambda x: "|".join([i['name'] for i in x]) if isinstance(x, list) else None)
</code></pre>

<h3>5. Cleaning Numerical Columns</h3>
<ul>
    <li><strong>Objective:</strong> Convert columns like <code>budget</code>, <code>id</code>, and <code>popularity</code> to numeric types, and handle invalid values.</li>
    <li><strong>Code Snippet:</strong></li>
</ul>
<pre><code>df["budget"] = pd.to_numeric(df["budget"], errors="coerce").replace(0, None)
df["revenue"] = pd.to_numeric(df["revenue"], errors="coerce").replace(0, None)
</code></pre>

<h3>6. Cleaning DateTime Columns</h3>
<ul>
    <li><strong>Objective:</strong> Convert the <code>release_date</code> column to datetime format and handle invalid dates.</li>
    <li><strong>Code Snippet:</strong></li>
</ul>
<pre><code>df["release_date"] = pd.to_datetime(df["release_date"], errors="coerce")
</code></pre>

<h3>7. Cleaning Text/String Columns</h3>
<ul>
    <li><strong>Objective:</strong> Identify and replace non-standard missing values in text columns like <code>overview</code> and <code>tagline</code>.</li>
    <li><strong>Code Snippet:</strong></li>
</ul>
<pre><code>df["overview"] = df["overview"].replace(["No overview found.", "No Overview", "No movie overview available.", " ", "No overview yet."], None)
df["tagline"] = df["tagline"].replace("-", None)
</code></pre>

<h3>8. Removing Duplicates</h3>
<ul>
    <li><strong>Objective:</strong> Identify and remove duplicate rows in the dataset.</li>
    <li><strong>Code Snippet:</strong></li>
</ul>
<pre><code>df.drop_duplicates(inplace=True)
df.drop_duplicates(subset="id", inplace=True)
</code></pre>

<h3>9. Handling Missing Values & Removing Observations</h3>
<ul>
    <li><strong>Objective:</strong> Drop rows with missing <code>id</code> or <code>title</code>, and retain rows with at least 10 non-NaN values.</li>
    <li><strong>Code Snippet:</strong></li>
</ul>
<pre><code>df.dropna(subset=["id", "title"], inplace=True)
df.dropna(thresh=10, inplace=True)
</code></pre>

<h3>10. Final Cleaning Steps</h3>
<ul>
    <li><strong>Objective:</strong> Keep only movies with status "Released", reorder columns, reset the index, and save the cleaned dataset.</li>
    <li><strong>Code Snippet:</strong></li>
</ul>
<pre><code>df = df.loc[df["status"] == "Released"].copy()
df.drop(columns=["status"], inplace=True)
df.reset_index(drop=True, inplace=True)

df.to_csv("movies_clean.csv", index=False)
</code></pre>

<h2>Final Output</h2>
<p>The cleaned dataset is saved as <code>movies_clean.csv</code>, ready for further analysis.</p>

</body>
</html>
