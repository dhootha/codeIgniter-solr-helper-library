# Codeigniter solr Query URL generator#

I start program with Codeigniter framework recently and i fall in love with it, specially the [Active Record]. So I decided to use Codeigniter in my current Web Project.

As we know The [Active Record] is built in CI framework for doing CURD with mysql database, but most of data is come from solr rest api in my own Project. I could not find any good Library for solr api so I have to make it by myself.

Current Library is version 2, solr api is very powerful so I only covered most of mine own functions for using solr. I will continually update this Library until it meet my own requirements.

##New helper updated!!
I updated a helper to help CI user parse solr results in JSON/PHP formats. 

**Parse json format**

arg1 = solr query url;arg2 = facet array
	
	solrResult_json_parser($url,$facetinput)
	
**Return array() with 'count','results','facet'**
	
**Parse PHP format**

arg1 = solr query url;arg2 = facet array
	
	solrResult_php_parser($url,$facetinput)

**Return array() with 'count','results','facet'**


###Parse exmple with json object###
	
	//variables
	$url = $this->solr_url->echo_solr_url('json',"0","20");//Query url created from solr_url.php class
	$facetArr = array('IBType_t','InstClassified_s');//which feild we need for put into facet	
	
	//Parse json object
	$solr = solrResult_json_parser($url,$facetArr);
	
	//get count value
	$count = $solr['count'];
	
	//get results
	$results = $solr['results'];//json object
	
	//example print out all the english name
	foreach ($results as $key => $value) {
		echo $value->EnName_t.'<br>';
	}
	
	//get facet 
	$facet = $solr['facet'];//array()
	
	//example print out facet with number
	foreach ($facet as $key => $value) {
		echo $value[0].":".$value[1]."<br>";
	}
	
**Output: Print out message**

	**JSON parser**
	**Number of results:**
	7
	**Solr results with EnName_t:**
	Chinese Academy of Science CAS (1)
	Library of the Chinese Academy of Sciences
	Chinese Academy of Agricultural Sciences
	Graduate School of the Chinese Academy of Science
	Jiangsu Academy of Agricultural Sciences
	China Academy of Traditional Chinese Medicine
	Academy of Mathematics and System Sciences CAS
	**Facet information:**
	Webometrics:7
	研究院/所/中心:2

## Simple Query ##
### 1 Config "Host" "Port" and "Database name" ###
	get_base_url($config)
Sometime I need to use defferent solr API from defferent host so I need some sort of way to config it. if you only use one API, you should put $solr_conf in config.php to make it as a global variable
#### Example ####

	//input varialbe for get_base_url()
	$solr_conf = array('host' => '10.0.15.xx' , 'port' => '8080' , 'DB' => 'myDatebase');
	//get_base_url method will create base url for your query
	$this->solr_url->get_base_url($solr_conf);
	
	
	
### 2 create solr query ###
	getQuery($input,$operator)
$input is a array contains solr field($key) and keyword($value), $operatro is sting with 'AND'/'OR'
#### Example ####
	//input variable for getQuery()
	$inputArr = array('EnName_t' => 'Academy' , 'Country_t' => 'China' );
	//create solr query
	$this->solr_url->getQuery($inputArr,'AND');

### 3 create solr URL ###
	echo_solr_url($format,$offset = "0",$limits = "10")
$format = 'XML'/'JSON'/'PHP'/'PYTHON', $offset and $limits are optional arguments, default is 10 records
#### Example ####
	$this->solr_url->echo_solr_url('json'); //default 10 records in json format
	$this->solr_url->echo_solr_url('json',"0","20"); // from 0 - 20 records in json format

	
### Full example for Simple Query ###
	class Contraller extends CI_Controller {	
		function Institute () {
			parent::__construct();
			$this->load->library('solr_url');
		}
		
	public function myFunc(){
		//varables
		$solr_conf = array('host' => '10.0.15.xx' , 'port' => '8080' , 'DB' => 'myDatabase');
		$inputArr = array('EnName_t' => 'Academy' , 'Country_t' => 'China' );
		
	
		//call methods	
		$this->solr_url->get_base_url($solr_conf);
		$this->solr_url->getQuery($inputArr,'AND');
		echo $this->solr_url->echo_solr_url('json',"0","20");
	}
	
**Output**
	
	http://10.0.15.xx:8080/myDatabase/select/?q=EnName_t:Academy%20AND%20Country_t:China&wt=json&version=2.2&indent=on&start=0&rows=20

		
## Advanced Query ##

### 1 sort result ###
	getSort($sort,$order)
$sort = field name, $order = 'desc'/'asc'

#### Example ####
	
	$this->solr_url->getSort('EnName_t','desc');
**Output**

	&sort=EnName_t%20desc


### 2 facet ###
	getFacet($facetInput,$sort = Null,$miniCount = '1')
$facetInput = array('facet field1','facet field2'), $sort = array(facet feild => 'sort'), $miniCount = what is minimum number of facet, default number is 1

#### Example ####
	
	//input variable for getFacet()
	$facetArr = array('IBType_t','InstClassified_s');
	$this->solr_url->getFacet($facetArr,array('IBType_t'=>'index'),'3');

**Output**
			
	&facet=on&facet.field=IBType_t&facet.field=InstClassified_s&facet.mincount=3&fIBType_tfacet.sort=index
	
### 3 fq:Search within query results ###
	public function getFQ($fqInput)
$fqInput is a array() contains solr field($key) and keyword($value)
#### Example ####
	$this->solr_url->getFQ(array('ESI_Fields14_S'=>'Engineering'));
	
	
### Full example for Advanced Query ###
	class Contraller extends CI_Controller {	
		function Institute () {
			parent::__construct();
			$this->load->library('solr_url');
		}
		
	public function myFunc(){
		//varables
		$solr_conf = array('host' => '10.0.15.xx' , 'port' => '8080' , 'DB' => 'myDatabase');
		$inputArr = array('EnName_t' => 'Academy' , 'Country_t' => 'China' );
		$facetArr = array('IBType_t','InstClassified_s');
	
		//call methods	
		$this->solr_url->get_base_url($solr_conf);
		$this->solr_url->getQuery($inputArr,'AND');
		$this->solr_url->getSort('EnName_t','desc');
		$this->solr_url->getFacet($facetArr,array('IBType_t'=>'index'),'3');
		$this->solr_url->getFQ(array('ESI_Fields14_S'=>'Engineering'));
		
		//create URL now!!
		echo $this->solr_url->echo_solr_url('json',"0","20");
	}

**Output**
	
	http://10.0.15.xx:8080/myDatabase/select/?q=EnName_t:Academy%20AND%20Country_t:China&wt=json&version=2.2&indent=on&start=0&rows=20&sort=EnName_t%20desc&facet=on&facet.field=IBType_t&facet.field=InstClassified_s&facet.mincount=3&fIBType_tfacet.sort=index&fq=ESI_Fields14_S:Enginerring"



	
	
	
	
	
	
	
	
	
	
	
	
	
	
	


---------

[Active Record]: http://en.wikipedia.org/wiki/Active_record_pattern
[package_control]: http://wbond.net/sublime_packages/package_control
