Persisting the computed data is done by developing the data source that computes the data (usually an `SqlQueryable`),
and saving the result into the database table.

Rhetos contains concepts that help automate the implementation:
`KeepSynchronized`, `ChangesOnLinkedItems`, `ChangesOnChangedItems`, `ChangesOnBaseItem` and `ChangesOnReferenced`.

## Examples

The following examples are taken from [Bookstore](https://github.com/Rhetos/Bookstore) demo application, available on GitHub.

Example 1:

    Module Bookstore
    {
        // ComputeBookInfo computes some information about the book by using SQL query.
        // The result is persisted (as a cache) in Entity BookInfo, and updated automatically.

        SqlQueryable ComputeBookInfo
            "
                SELECT
                    b.ID,
                    NumberOfComments = COUNT(bc.ID)
                FROM
                    Bookstore.Book b
                    LEFT JOIN Bookstore.Comment bc ON bc.BookID = b.ID
                GROUP BY
                    b.ID
            "
        {
            Extends Bookstore.Book;
            Integer NumberOfComments;

            ChangesOnLinkedItems Bookstore.Comment.Book;
        }

        Entity BookInfo
        {
            ComputedFrom Bookstore.ComputeBookInfo
            {
                AllProperties;
                KeepSynchronized;
            }
        }
    }

Example 2:

    Module Bookstore
    {
        // ComputeBookRating computes some information about the book by using C# implementation from an external dll.
        // The result is persisted (as a cache) in Entity BookRating, and updated automatically.

        Computed ComputeBookRating 'repository =>
            {
                var allBooksIds = repository.Bookstore.Book.Query().Select(b => b.ID).ToArray();
                return this.Load(allBooksIds).ToArray();
            }'
        {
            Extends Bookstore.Book;
            Decimal Rating;

            FilterBy 'IEnumerable<Guid>' '(repository, booksIds) =>
                {
                    var ratingInput = repository.Bookstore.Book.Query(booksIds)
                        .Select(b =>
                            new Bookstore.Algorithms.RatingInput
                            {
                                BookId = b.ID,
                                Title = b.Title,
                                IsForeign = b.Extension_ForeignBook.ID != null
                            });

                    var ratingSystem = new Bookstore.Algorithms.RatingSystem();
                    var ratings = ratingSystem.ComputeRating(ratingInput);

                    return ratings.Select(rating => new ComputeBookRating { ID = rating.BookId, Rating = rating.Value }).ToArray();
                }';
            
            ChangesOnBaseItem;
            ChangesOnChangedItems Bookstore.ForeignBook 'IEnumerable<Guid>' 'changedItems => changedItems.Select(fb => fb.ID)';
        }

        ExternalReference 'Bookstore.Algorithms.RatingSystem, Bookstore.Algorithms'; // One class per assembly is enough for external reference.

        Entity BookRating
        {
            ComputedFrom Bookstore.ComputeBookRating
            {
                AllProperties;
                KeepSynchronized;
            }
        }
    }

## See also

* [Bookstore](https://github.com/Rhetos/Bookstore) demo application
