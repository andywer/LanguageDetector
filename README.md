LanguageDetector [![Build Status](https://travis-ci.org/andywer/language-detector.png)](https://travis-ci.org/andywer/language-detector)
================

PHP library to detect languages from any free text.

It follows the approach described in the [paper](http://scholar.google.com.py/scholar?q=N-Gram-Based+Text+Categorization), a given text is tokenized into [N-Grams](http://en.wikipedia.org/wiki/N-gram) (we cleanup whitespaces before doing this step). Then we sort the `tokens` and we compare against a language `model`.

*Fork of [crodas/languagedetector](https://github.com/crodas/LanguageDetector), since the original package seems abandoned.*

How it works
------------

The first thing we need is a `language model` (which looks like [this file](https://github.com/crodas/LanguageDetector/blob/master/example/datafile.php)) that is used to compare the texts against at classification time. This process must done *before* anything, and it can be generated with an script similar to [this file](https://github.com/crodas/LanguageDetector/blob/master/example/learn.php).

```php
// register the autoloader
require 'lib/LanguageDetector/autoload.php';

// it could use a little bit of memory, but it's fine
// because this process runs once.
ini_set('memory_limit', '1G');

// we load the configuration (which will be serialized
// later into our language model file
$config = new LanguageDetector\Config;

$c = new LanguageDetector\Learn($config);
foreach (glob(__DIR__ . '/samples/*') as $file) { 
    // feed with examples ('language', 'text');
    $c->addSample(basename($file), file_get_contents($file));
}

// some callback so we know where the process is 
$c->addStepCallback(function($lang, $status) {
    echo "Learning {$lang}: $status\n";
});

// save it in `datafile`. 
// we currently support the `php` serialization but it's trivial
// to add other formats, just extend `\LanguageDetector\Format\AbstractFormat`. 
//You can check example at https://github.com/crodas/LanguageDetector/blob/master/lib/LanguageDetector/Format/PHP.php
$c->save(AbstractFormat::initFormatByPath('language.php'));
```

Once we have our language model file (in this case `language.php`) we're ready to classify texts by their language.

```php
// register the autoloader
require 'lib/LanguageDetector/autoload.php';

// we load the language model, it would create
// the $config object for us.
$detect = LanguageDetector\Detect::initByPath('language.php');

$lang = $detect->detect("Agricultura (-ae, f.), sensu latissimo, 
est summa omnium artium et scientiarum et technologiarum quae de 
terris colendis et animalibus creandis curant, ut poma, frumenta, 
charas, carnes, textilia, et aliae res e terra bene producantur. 
Specialius, agronomia est ars et scientia quae terris colendis student, 
agricultio autem animalibus creandis.")

var_dump($lang);
```

And that's it.

Algorithms
----------

The project is designed to work with modules, which means you can provide your own algorithm for `sorting` and `comparing` the N-Grams. By default the library implements the [PageRank](http://en.wikipedia.org/wiki/PageRank) as `sorting` algorithm, and *out of place* (described in the paper) as `comparing`. 

In order to supply your own algorithms, you must change the `$config` at *learning stage* to load your own classes (which by the way should implement some interaces).

Language Detection Training Files
---------------------------------

Have a look at `example/samples` directory. For more advanced traning data, visit the [Leipzig Corpora Download Page](http://corpora2.informatik.uni-leipzig.de/download.html).

Languages with non-latin characters
-----------------------------------

Remember to set the Config's `mb` property (already before creating the language model) if you train for languages based on non-latin characters. Use UTF-8 encoded texts.
