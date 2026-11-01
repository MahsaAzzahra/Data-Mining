Ekstraksi Data Film Star Wars dengan rvest
================
Mahsa Azzahra

Dokumen ini memuat proses ekstraksi data film Star Wars dari halaman
HTML statis (<https://rvest.tidyverse.org/articles/starwars.html>)
menggunakan bahasa R dan paket `rvest`. Halaman tersebut memuat tujuh
film, dan setiap film berada dalam satu elemen `section`. Data yang
diekstraksi meliputi judul, tanggal rilis, sutradara, dan teks pembuka
(opening crawl) dari setiap film.

Alur kerja yang digunakan mengikuti materi ekstraksi web statis: memuat
pustaka, membaca HTML, menyeleksi node, mengekstrak teks, memeriksa
hasil, dan menyimpan hasil ke CSV.

## A. Persiapan

Pustaka `tidyverse` digunakan untuk membentuk tibble serta mengolah teks
(`str_remove`, `str_squish`, `parse_date`), sedangkan pustaka `rvest`
digunakan untuk membaca dan mengekstraksi data dari HTML.

``` r
library(tidyverse)
library(rvest)
```

## B. Membaca Dokumen HTML

Fungsi `read_html()` mengunduh dan menguraikan berkas HTML dari URL yang
diberikan. Hasilnya disimpan dalam objek `html`.

``` r
html <- read_html("https://rvest.tidyverse.org/articles/starwars.html")
```

## C. Menyeleksi Node `section`

Fungsi `html_elements()` menemukan semua elemen yang cocok dengan
selektor CSS. Pada halaman ini, setiap film berada dalam satu tag
`section`, sehingga selektor `"section"` menghasilkan satu node untuk
setiap film.

``` r
sections <- html |> html_elements("section")
length(sections)
```

    ## [1] 7

Hasil di atas menunjukkan bahwa terdapat tujuh node `section`, sesuai
dengan jumlah film pada halaman.

## D. Ekstraksi Data ke dalam Tibble

Setiap kolom diekstrak dari node `section` dengan fungsi
`html_element()` (bentuk tunggal). Fungsi ini selalu menghasilkan satu
output untuk setiap input, dan menghasilkan `NA` jika tidak ada
kecocokan, sehingga jumlah baris data tetap terjaga. Penjelasan setiap
kolom adalah sebagai berikut.

1.  `title`: diambil dari tag `h2` dan diubah menjadi teks dengan
    `html_text2()`.
2.  `released`: diambil dari tag `p` pertama pada setiap `section`. Teks
    `"Released: "` dihapus dengan `str_remove()`, kemudian
    `parse_date()` mengubahnya menjadi tipe tanggal.
3.  `director`: diambil dari elemen berkelas `.director`.
4.  `intro`: diambil dari elemen berkelas `.crawl`. Fungsi
    `str_squish()` digunakan untuk mengubah baris kosong antarparagraf
    menjadi satu spasi, sehingga teks setiap film menempati satu baris
    pada berkas CSV.

``` r
starwars_data <- tibble(
  title = sections |>
    html_element("h2") |>
    html_text2(),
  released = sections |>
    html_element("p") |>
    html_text2() |>
    str_remove("Released: ") |>
    parse_date(),
  director = sections |>
    html_element(".director") |>
    html_text2(),
  intro = sections |>
    html_element(".crawl") |>
    html_text2() |>
    str_squish()
)
```

## E. Pemeriksaan Hasil Ekstraksi

Struktur data diperiksa dengan `glimpse()` untuk memastikan jumlah
baris, jumlah kolom, dan tipe setiap kolom sudah sesuai.

``` r
glimpse(starwars_data)
```

    ## Rows: 7
    ## Columns: 4
    ## $ title    <chr> "The Phantom Menace", "Attack of the Clones", "Revenge of the…
    ## $ released <date> 1999-05-19, 2002-05-16, 2005-05-19, 1977-05-25, 1980-05-17, …
    ## $ director <chr> "George Lucas", "George Lucas", "George Lucas", "George Lucas…
    ## $ intro    <chr> "Turmoil has engulfed the Galactic Republic. The taxation of …

Kolom judul, tanggal rilis, dan sutradara ditampilkan dalam bentuk
tabel.

``` r
knitr::kable(starwars_data[, c("title", "released", "director")])
```

| title                   | released   | director         |
|:------------------------|:-----------|:-----------------|
| The Phantom Menace      | 1999-05-19 | George Lucas     |
| Attack of the Clones    | 2002-05-16 | George Lucas     |
| Revenge of the Sith     | 2005-05-19 | George Lucas     |
| A New Hope              | 1977-05-25 | George Lucas     |
| The Empire Strikes Back | 1980-05-17 | Irvin Kershner   |
| Return of the Jedi      | 1983-05-25 | Richard Marquand |
| The Force Awakens       | 2015-12-11 | J. J. Abrams     |

Contoh isi kolom `intro` pada film pertama:

``` r
starwars_data$intro[1]
```

    ## [1] "Turmoil has engulfed the Galactic Republic. The taxation of trade routes to outlying star systems is in dispute. Hoping to resolve the matter with a blockade of deadly battleships, the greedy Trade Federation has stopped all shipping to the small planet of Naboo. While the Congress of the Republic endlessly debates this alarming chain of events, the Supreme Chancellor has secretly dispatched two Jedi Knights, the guardians of peace and justice in the galaxy, to settle the conflict…."

## F. Penyimpanan Hasil ke CSV

Data yang sudah rapi disimpan ke format CSV dengan `write.csv()`.
Argumen `row.names = FALSE` mencegah terbentuknya kolom indeks tambahan.

``` r
write.csv(starwars_data, "starwars_data.csv", row.names = FALSE)
```

Berkas `starwars_data.csv` tersimpan di working directory yang sedang
aktif.

## G. Kesimpulan

Ekstraksi berhasil mengambil data tujuh film Star Wars ke dalam satu
tibble berisi empat kolom, yaitu `title`, `released`, `director`, dan
`intro`. Seluruh selektor bekerja dengan benar, sehingga tidak terdapat
nilai `NA` pada data hasil ekstraksi.
