# SearchAlgo
A lightweight Python based text search framework for Django.

SearchAlgo uses nltk to clean data and gensim to pickup keywords from longer text fields. This information is stored in a search_algo_condense text field in the database allowing for speedy and accurate search results.

## Usage
* Add import in your `models.py` file: `from search_algo_app.algo import SearchAlgoModel`.
* Models using SearchAlgo should be a subclass of SearchAlgoModel.
* Setup models and setting variables.
```python 
from search_algo_app.algo import SearchAlgoModel

class MyModel(SearchAlgoModel):
    field1 = models.CharField(max_length=256)
    field2 = models.CharField(max_length=256)
    field3 = models.TextField()

    non_summarized_search_fields = ['field1', 'field2']
    summarized_search_fields = ['field3']
```
* Process search_algo_condense:
```bash
python manage.py shell
from <app_name>.models import <model> # Replace with your own app name and model name.
<model>.process_condense() # Usually takes less than 0.05 seconds per database entry.
```
* `<model>.search(<query>)` returns a QuerySet for the search.
```python
def my_view(request):
    dataset = MyModel.objects.all()

    if 'search' in request.GET:
        query = request.GET['search']
        dataset = AllTheNews.search(query)

    return render(request, 'myapp/mytemplate.html', {'dataset': dataset})

```

## Settings
All settings are defined as variables in the model.

| variable name | type | description | required | default |
| -- | -- | -- | --| -- |
| `non_summarized_search_fields` | list of strings | Shorter, more relevant model text fields which are stored in full in the `search_algo_condense` field used by the database for regular searches. | Yes | None |
| `summarized_search_fields` | list of strings | Longer model text fields which are processed for keywords to be stored in the `search_algo_condense` field. | No | None |
| `spell_check` | boolean | Defines if queries are checked and corrected for spelling. | No | False |

## Testing with Sample AllTheNews Dataset
Sample data compiled using [All the news dataset](https://www.kaggle.com/snapcrack/all-the-news) by Andrew Thompson.

* Download [sample dataset json file](https://mega.nz/file/smpgwS5Z#mVIeyQbqC8Sv4A2w5balkXtOfwCXG4QwX8zq-xqFiKc).
* Run python `python manage.py makemigrations` and `python manage.py migrate`.
* Run `python manage.py loaddata <path to sample_all_the_news.json>`.
* Process search_algo_condense:
```shell
python manage.py shell
from <app_name>.models import <model> # Replace with your own app name and model name.
<model>.process_condense() # Usually takes less than 0.05 seconds per database entry.
```
* Run `python manage.py runserver`.
* Test out at [localhost:8000/all_the_news_dataset](http://localhost:8000/all_the_news_dataset/).

## Screenshots
A specific search returns only few results.
![](/screenshots/specific_search.png)

A vague search would return a lot of results.
![](/screenshots/vague_search.png)

Exact searches using `--exact` can be used to fetch very specific entries.
![](/screenshots/exact_search.png)

Misspelled searches should work as expected with `spell_check = True`.
![](/screenshots/misspelled_search.png)

Misspelled searches with the `--exact` flag will usually result in 0 results.
![](/screenshots/exact_misspelled_search.png)
