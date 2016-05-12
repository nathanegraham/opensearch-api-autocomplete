##Autocomplete example with Wikipedia data using Opensearch API

The below script is an example of how to autocomplete a web form using the Opensearch API. In this example, the form includes seven fields that are populated with biographical data from Wikipedia. This example uses Formidable (a WordPress form builder) but could be used any web form with some small tweaks.

This example populates author info for a form using Wikipedia parameters based upon the selection from the automcomplete field.  For example, if the user searches Ernest Hemingway, it autocompletes based upon the existing Wikipedia article. Once the author name is selected, it autopopulates relevant fields from the Wikipedia article.

![alt text](http://cdn.digitalborn.org/wp-content/uploads/2016/05/wikipedia_opensearch_api_formidable_integration.gif?0d1a96 "Example of autocomplete")

For a quick background on how autocomplete works with Wikipedia Opensearch API see [this article](http://w3lessons.info/2015/03/01/autocomplete-search-using-wikipedia-api-and-jquery-ui/). You can find the parameters for the Opensearch API [here](https://www.mediawiki.org/wiki/API:Opensearch).

###Setup Your Form
* Click on Forms -> Add New
* Add seven single line text fields to your form: Name, Birth Name, Birth Place, Birth Date, Death Date, Author Image, Author Link
* Click Create

###Integrating with Opensearch API
* Note the field ids of the seven fields you created
* Note that you will replace field_entrykey and form_key with your relevant field and form ids or keys. 
* The searchInput field should be your search box. All of the other parameters will populate based upon what is selected in the searchInput field.

```
<script src="https://ajax.googleapis.com/ajax/libs/jquery/2.1.3/jquery.min.js"></script>
<link rel="stylesheet" href="https://ajax.googleapis.com/ajax/libs/jqueryui/1.11.3/themes/smoothness/jquery-ui.css" />
<script src="https://ajax.googleapis.com/ajax/libs/jqueryui/1.11.3/jquery-ui.min.js"></script>
<style>
#form_key25{
transition: opacity 200ms
}
#form_key25.loading{
opacity:0.5;
}
</style>
<script>
jQuery(function($){
var inputsMap = {
birthName: "#field_entrykey2",
birthPlace: "#field_entrykey3",
birthDate: "#field_entrykey4",
deathDate: "#field_entrykey5",
link: "#field_entrykey6",
photo: "#field_entrykey7",

},
valuesMap = {},
$searchInput = $("#field_entrykey1"),
$form = $("#form_key25");

$searchInput.autocomplete({
source: function(request, response) {
$.get("http://en.wikipedia.org/w/api.php?action=opensearch&format=json",
{search: request.term},
function(data) {

valuesMap = {};
data[1].forEach(function(val, idx){
valuesMap[val] = data[3][idx].replace(/^.+?\/([^/]+$)/, "$1")
});

response(data[1]);
}, "jsonp");

},
select:function(event, ui){
getContent(valuesMap[ui.item.value]);
}
});
function getContent(title ){
var result = {},
match;

setLoader(true);

$.get("https://en.wikipedia.org/w/api.php?action=query&prop=revisions&rvprop=content&&format=json&redirects",
{titles: title},
function(resp){
//parse content

var content = JSON.stringify(resp).replace(/<!--[\s\S]+?-->/g, ''),
result = {
name: (content.match(/"title":"(.+?)"/) || [])[1] || "",
birthName: (content.match(/\|\s*birth_name\s*=\s*(.*?)\\n\|/)|| [])[1] || "",
birthPlace: (content.match(/\|\s*birth_place\s*=\s*(.*?)\\n\|/)|| [])[1] || "",
birthDate: (content.match(/\|\s*birth_date\s*=\s*(.*?)\\n\|/)|| [])[1] || "",
deathDate: (content.match(/\|\s*death_date\s*=\s*(.*?)\\n\|/)|| [])[1] || "",
photo: (content.match(/\|\s*image\s*=\s*(.*?)(\\n|{{)/)|| [])[1] || "",
};
if(!result.photo){
result.photo = (content.match(/\|\s*image1\s*=\s*(.*?)(\\n|{{)/)|| [])[1] || ""
}
if(result.birthPlace){
var birthPlace = result.birthPlace.replace(/\[\[(.+?\|)?(.+?)\]\]/g, '$2'),
chunks = birthPlace.split(", ");

//result.country = chunks.pop()
result.birthPlace = chunks.join(", ")

}

if(result.birthDate){
match = result.birthDate.match(/\|(\d\d\d\d)\|(\d\d?)\|(\d\d?)/)
if(match){
result.birthDate = match[2] + "/" + match[3] + "/" + match[1]
}

}

if(result.deathDate){
match = result.deathDate.match(/\|(\d\d\d\d)\|(\d\d?)\|(\d\d?)/)
if(match){
result.deathDate = match[2] + "/" + match[3] + "/" + match[1]
}

}

result.link = "https://en.wikipedia.org/wiki/"+title

getImage(result.photo, function(photo){
if(photo) result.photo = photo;
setLoader(false);
//set fields
$.each(result, function(key, val){
$(inputsMap[key]).val(val);
})
})
}
,
"jsonp"
).error(function(e){
setLoader(false);
console.log('err', e)
});

}

function getImage(fileName, callback){
if(!fileName) return callback();

$.get("http://en.wikipedia.org/w/api.php?action=query&format=json&prop=imageinfo&iiprop=url",
{titles: "File:"+fileName},
function(data) {
var pages = $.map(data.query.pages, function(val, key) { return val; }),
page = pages[0];

if(!page || !page.imageinfo || !page.imageinfo.length ) return callback();

callback(page.imageinfo[0].url);

}, "jsonp")
.error(function(e){
callback()
});
}

function setLoader(active){
$form[active?'addClass':'removeClass']('loading');
}

});
</script>
```

###Add to Form
* Click on Forms
* Select your form and select Settings -> Customize HTML -> After Fields
* Add your script to After Fields
![alt text](http://cdn.digitalborn.org/wp-content/uploads/2016/05/Screen-Shot-2016-05-01-at-10.36.05-PM-1024x151.png "After Fields")

If you’re looking to do the above with your own form and have particular parameters you would like to populate, you can use the  [WikiMedia API sandbox](https://www.mediawiki.org/wiki/Special:ApiSandbox) to start finding your parameters.
