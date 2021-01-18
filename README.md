# MultiSearchBundle

This bundle provides basic service and form type for Multi Search in Doctrine. 

[![Build Status](https://scrutinizer-ci.com/g/petkopara/PetkoparaMultiSearchBundle/badges/build.png?b=master)](https://scrutinizer-ci.com/g/petkopara/PetkoparaMultiSearchBundle/build-status/master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/petkopara/PetkoparaMultiSearchBundle/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/petkopara/PetkoparaMultiSearchBundle/?branch=master)
[![SensioLabsInsight](https://insight.sensiolabs.com/projects/462874f8-228d-4d9c-951e-e5c001a41c46/mini.png)](https://insight.sensiolabs.com/projects/462874f8-228d-4d9c-951e-e5c001a41c46)
[![Latest Stable](https://img.shields.io/packagist/v/petkopara/multi-search-bundle.svg?maxAge=2592000?style=flat-square)](https://packagist.org/packages/petkopara/multi-search-bundle)
[![Total Downloads](https://img.shields.io/packagist/dt/petkopara/multi-search-bundle.svg?maxAge=2592000?style=flat-square)](https://packagist.org/packages/petkopara/multi-search-bundle)


## Description
Search in all of entity columns by given search term. 
In response returns `Doctrine\ORM\QueryBuilder` containing the multiple search criteria. 
The searched columns can be specified. 
    

## Installation 


### Using composer

    composer require petkopara/multi-search-bundle

Add it to the `AppKernel.php` class:

    new Petkopara\MultiSearchBundle\PetkoparaMultiSearchBundle(),


##Usage

### Service
You can directly use the service and to apply the multi search to any doctrine query builder.

    public function indexAction(Request $request)
    {
        $search = $request->get('search');
        $em = $this->getDoctrine()->getManager();
        
        $qb = $em->getRepository('AppBundle:Post')->createQueryBuilder('e');
        $qb = $this->get('petkopara_multi_search.builder')->searchEntity($qb, 'AppBundle:Post', $search);
       //$qb = $this->get('petkopara_multi_search.builder')->searchEntity($qb, 'AppBundle:Post', $search, array('name', 'content'), 'wildcard');
    
        ..
    }

### Form

Create your form type and include the multiSearchType in the buildForm function: 

    use Petkopara\MultiSearchBundle\Form\Type\MultiSearchType;

    public function buildForm(FormBuilderInterface $builder, array $options)
    {
        $builder
                ->add('search', MultiSearchType::class, array(
                    'class' => 'AppBundle:Post', //required
                    'search_fields' => array( //optional, if it's empty it will search in the all entity columns
                        'name',
                        'content'
                     ), 
                     'search_comparison_type' => 'wildcard' //optional, what type of comparison to applied ('wildcard','starts_with', 'ends_with', 'equals')
                     
                ))
        ;
    }

In the controller add call to the multi search service: 

    public function indexAction(Request $request)
    {
        $search = $request->get('search');
        $em = $this->getDoctrine()->getManager();
        $queryBuilder = $em->getRepository('AppBundle:Post')->createQueryBuilder('e');
        $filterForm = $this->createForm('AppBundle\Form\PostFilterType');

        // Bind values from the request
        $filterForm->handleRequest($request);

        if ($filterForm->isValid()) {
            // Build the query from the given form object
            $queryBuilder = $this->get('petkopara_multi_search.builder')->searchForm($queryBuilder, $filterForm->get('search'));
        }
        
        ..
    }

Render your form in the view 

    {{ form_rest(filterForm) }}


## Available Options

The provided type has 2 options:

* `search_fields` - array of the entity columns that will be added in the search. If it's not set then will search in all columns
* `search_comparison_type` -  how to compare the search term.   

  * `wildcard` - it's equivalent to the %search% like search.

  * `equals` - like operator without wildcards. Wildcards still can be used with `equals` if the search term contains *.

  * `starts_with` - it's equivalent to the %search like search.
  
  * `ends_with` - it's equivalent to the search% like search.
  
These parameters can be applyed to the service as well as 4th and 5th parameter to `searchEntity()` method

## Author

Petko Petkov - petkopara@gmail.com


## License

MultiSearchBundle is licensed under the MIT License.



