Config loader for PHP
=====================

A configuration loader with support for YAML, TOML (0.2.0) and JSON.

Inspired by [Yosymfony\ConfigServiceProvider](https://github.com/yosymfony/ConfigServiceProvider).

Installation
------------

Use [Composer](http://getcomposer.org/) to install Config-loader package:

Add the following to your `composer.json` and run `composer update`.

    "require": {
        "yosymfony/config-loader": "1.*@dev"
    }

More informations about the package on 
[Packagist](https://packagist.org/packages/yosymfony/config-loader).

Usage
-----

```
use Symfony\Component\Config\FileLocator;

class MyClass
{
    public function setUp()
    {
        // Set up the paths
        $locator = new FileLocator(array('/path-to-your-files-1', '/path-to-your-files-2'));

        // Set up the config loader 
        $config = new Config(array(
            new TomlLoader($locator),
            new YamlLoader($locator),
            new JsonLoader($locator),
        ));
    }
}
```
    
### Load a configuration file:

    $config->load('user.yml');
    
    // or load with absolute path:
    $config->load('/var/config/user1.yml');
    
#### *.dist* files

This library have support to `.dist` files. The location of a file follow the next hierachy:

- filename.ext
- filename.ext.dist (if filename.ext not exists)

### Load inline configuration:
    
    $repository = $config->load('server: "mail.yourname.com"', Config::TYPE_YAML);
    // or
    $repository = $config->load('server = "mail.yourname.com"', Config::TYPE_TOML);
    
Repository
----------
The configuration file is loaded to a repository. A repository is a wrapper with 
array access interface that supply methods for validating configurations values 
and merge with other repositories.

    $repository->get('name', 'noname'); // If 'name' not exists return 'noname'
    $repository['name']; // Get the element in 'name' key

### Validating configurations
The values and the structure can be validated using a definition class from 
[Symfony Config component](http://symfony.com/doc/current/components/config/definition.html). 
For example, for this configuratión file:

    # Yaml file
    port: 25
    server: "mail.yourname.com"

you can create the below definition:

    use Symfony\Component\Config\Definition\ConfigurationInterface;
    use Symfony\Component\Config\Definition\Builder\TreeBuilder;
    
    class MyConfigDefinitions implements ConfigurationInterface
    {
        public function getConfigTreeBuilder()
        {
            $treeBuilder = new TreeBuilder();
            $rootNode = $treeBuilder->root(0);
            
            $rootNode->children()
                ->integerNode('port')
                    ->end()
                ->scalarNode('server')
                    ->end()
            ->end();
            
            return $treeBuilder;
        }
    }

and check your repository: `$repository->validateWith(new MyConfigDefinitions());`
An exception will be thrown if any definition constraints are violated.

### Operations

#### Unions
You can get the union of repository A with other B with C as result: 
`$resultC = $repositoryA->union($repositoryB);`. 
The values of `$repositoryB` have less priority than `$repositoryA`.

#### Intersections
You can get the intersection of repository A with other B with C as result: 
`$resultC = $repositoryA->intersection($repositoryB);`. 
The values of `$repositoryB` have less priority than `$repositoryA`.

### Create a blank repository
Create a blank repository is too easy. You only need create a instance of 
`ConfigRepository` and use the array interface or set method to insert new values:

    use Yosymfony\Config-loader\Repository;
    
    //...
    
    $repository = new ConfigRepository();
    $repository->set('key1', 'value1');
    // or
    $repository['key1'] = 'value1';
    
and the next, you merge with others or validate it.

Unit tests
----------

You can run the unit tests with the following command:

    $ cd your-path/vendor/yosymfony/config-loader
    $ composer.phar install --dev
    $ phpunit