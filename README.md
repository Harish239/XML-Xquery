# XML and XQuery


## The following DTD data format A that describes the information about authors and books.

## DTD data format A:

```
<?xml version="1.0" encoding="UTF-8"?>

<!DOCTYPE biblio[

<!ELEMENT biblio (author*)>

<!ELEMENT author (name,book*)>

<!ELEMENT name (#PCDATA)>

<!ELEMENT book (title, category,rating,price)>

<!ELEMENT title (#PCDATA)>

<!ELEMENT category(#PCDATA)>

<!ELEMENT rating(#PCDATA)>

<!ELEMENT price (#PCDATA)>

<!ATTLIST book year CDATA #REQUIRED>

]>
```

## 1. Find the names of all Jeff’s co-authors and list them together with the titles of books that were co-authored.
```
for $jeff in doc("/db/books.xml")/biblio/author
where $jeff/name='Jeff'
for $title in $jeff/book/title
for $author in doc('books')/biblio/author
where $author/name!='Jeff' and $author/book/title = $title
return
<book>
{$title}
{$jeff/name}
{$author/name}
</book>
```

## 2. Return all the author pairs who have co-authored two or more books together, list their co-authored.
```
let $final :=(
let $author1 :=  doc("/db/books.xml")/biblio/author
for $x in $author1
for $y in $x/following-sibling::*
where $x/book/title = $y/book/title
for $title1 in $x/book
for $title2 in $y/book
where $title1/title = $title2/title
return <output>{$x/name}
{$y/name}
{$title1}</output>)
let $answer :=
(for $book1 in $final
for $book2 in $final
where deep-equal($book1/name, $book2/name) and  $book1/book/title!=$book2/book/title
return $book1)
return <coauthor>{$answer}</coauthor>
```

## 3. Find the average book price of each category and global. If a category has higher than global average book price, list one most expensive book and its authors, for each of those categories.
```let $authors := doc("/db/books.xml")/biblio/author
let $global_avg := avg($authors/book/price)
let $unique_category := distinct-values($authors/book/category)
let $categories :=(
for $category in $unique_category
let $avg := avg($authors/book[category = $category]/price)
return if($avg>$global_avg)
then (<category>{$category}</category>))
let $answer :=(
for $category in $categories
let $books_category := $authors/book[category = $category]
let $top_book :=(
for $book in $books_category
order by $book/price 
return $book)[1]
for $final_book in $top_book
let $names := (for $author in $authors
where $author/book/title = $final_book/title
return $author/name)
return <output>{$final_book/category}{$final_book/title}{$final_book/price}{$names}</output>)
return <result><categories>{$answer}</categories></result>
```

## 4. Return all the book title, book price and sort the book price from low to high.
```
let $unique_pairs :=
(let $books :=doc("/db/books.xml")//book
for $book in $books
group by $title:= $book/title, $price:=$book/price 
order by  xs:float($price)
return 
<book><title>{$title}</title>
<price>{$price}</price></book>)
for $book in $unique_pairs
return ($book/title, $book/price)
```
## 5. Return all the book title, book rating and sort the book rating from high to low.
```
let $unique_pairs :=
(let $books :=doc("/db/books.xml")//book
for $book in $books
group by $title:= $book/title, $rating:=$book/rating 
order by  xs:float($rating) descending
return 
<book><title>{$title}</title>
<rating>{$rating}</rating></book>)
for $book in $unique_pairs
return ($book/title, $book/rating)
```

## 6. Cheapest books in each category.
```
let $books :=doc("/db/books.xml")/biblio/author/book
let $categories := distinct-values($books/category)
for $category in $categories
let $book := $books[category = $category]
let $result:=(
for $b in $book
order by xs:float($b/price)
return $b)[1]
for $r in $result
return ($r/title,$r/price, $r/category)
```

## 7. Best Rating books.
```
let $books :=doc("/db/books.xml")/biblio/author/book
let $categories := distinct-values($books/category)
for $category in $categories
let $book := $books[category = $category]
let $result:=(
for $b in $book
order by xs:float($b/rating) descending
return $b)[1]
for $r in $result
return ($r/title,$r/rating, $r/category)
```

## 8. The text book requirement in this class is based on ‘category’: one ‘DB’, one ‘PL’, one ‘Science’, one ‘Others’. Return your plan for the book purchasing. The plan should assume you have $1800 how to get the best rating books.
```
let $author :=  doc("/db/books.xml")/biblio/author/book
let $dbs := $author[category = 'DB']
let $pls := $author[category = 'PL']
let $scs := $author[category = 'Science']
let $others := $author[category = 'Others']
let $result :=
(for $db in $dbs, $pl in $pls, $sc in $scs, $other in $others
let $rating_sum := $db/rating +$pl/rating+$sc/rating+$other/rating
return 
if(($db/price+$pl/price+$sc/price+$other/price)<=1800)
then
<combination>
{$db}
{$pl}
{$sc}
{$other}
<rating_sum>{$rating_sum}</rating_sum>
</combination>)
let $results :=(for $element in $result
order by  xs:float($element/rating_sum) descending
return $element)[1]
let $x :=(for $ele in $results/book
return ($ele/title,$ele/price,$ele/rating,$ele/category))
return $x
```

## 9. Define a DTD for an equivalent DTD data format B which stores the same information as A, but in which the authors are listed under their books. Write an XQuery query whose input is an XML document valid with respect to the DTD A and whose output is another XML document valid with respect to B.
```
let $authors :=doc("/db/books.xml")/biblio/author
let $unique_books := distinct-values($authors/book/title)
let $return_names :=(
for $book in $unique_books
let $rc := $authors/book[title =$book][1]
let $category := $rc/category
let $price := $rc/price
let $rating :=$rc/rating
let $year := $rc/@year
let $names := (for $author in $authors
where $author/book/title = $book
return $author/name)
return<output>
<book year ="{data($year[1])}">
<title>{$book}</title>
{$category[1]}
{$rating[1]}
{$price[1]}
{$names}
</book>
</output>)
return <biblio>
{$return_names}
</biblio>
```




