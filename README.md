# perl-stl

Perl Simple Templating Language Implementation 

Simple Template Language (stl) is purposely *almost* the simplest template language there can be whilst having some real world practical use just above straight up inserting of values into tags.
It is designed to save the effort of writing classes purely to template data with very minimal processing such as very few basic 'if' statements. 
It is NOT designed to excuse you from writing code where you should do. If this doesn't do what you want it to do, you should probably be writing some code instead.

The parser will accept a template and a hash/associative array of values, and do the following.

* Replace tags in the format {=tagname} with the value of the key of 'tagname' in the hash/associtive array.
* Replace tags in the format {=if tagname} some content {=end if} with the part of the template between the 'if' and 'end if' tags IF the tag exists and contains any content at all (including 0) 
* Allow very basic 'and' and 'or' rules to these 'if' statements. 
        Eg:  {=if tag1+tag2} We have both tag 1 and tag2 {=end if} 
        Eg2: {=if tag1|tag2} We have either tag 1 or tag 2 (and we might have both!) {=end if}
* Allow negation using '!' 
        Eg.  {=if tag1+!tag2} We have tag 1 but not tag 2 {=end if} 
        Eg2. {=if !tag1|tag2} We have either not got tag 1 or we have tag 2 {=end if}. 
        Note that the ! only applies to the immediately following tag and not the whole expression. 
* Allow basic equality testing. 
        Eg. {=if tag1=Yes}Value is 'yes'{=end_if}
        Eg2. {=if tag2=Value of tag 2} {=tag3} {=end_if} # insert tag3 if tag2 = 'Value of tag 2' (case sensitive)
* Combine many of the above rules (see ** for limitations below) 
        Eg:  {=if tagname+!other_tag+anothertag=thisvalue} {=value_of_yet_other_tag} and {=yet_another_tag_here} {=end_if}

Notes and limitations: 
----------------------
No parentheses or specified order of precedence for mixing and/or queries together. So Don't. 
No else/elseif, you'll need separate expressions.
+ and | are logical NOT mathematical operators so no on-the-fly addition etc.
Basic boolean testing HOWEVER: The ONLY false values by default are an empty string or a non-existent hash key (1) 
You CANNOT nest {=if} tags.
** - only ONE equality test per expression, which must be the last test you do (2) in the 'if' statement. 

Footnotes:
---------
(1) - the __hasValue specifically allows 0 to pass as a truthy value rather than the default false. Type is not taken into account so 0 and '0' are treated the same. If you want 0 to be false edit the __hasValue function
(2) - because it becomes far more complex to do this as a regex otherwise. Multiple splits on strings would work but this is not meant to be a whole language, just a very basic data templater. The fact that even one works when mixed with others was not planned just how the code came out.
(3) - Why should 0 evaluate to true by default? Because this was written for parsing database records, which often contained 0 as a field value that I wanted to print and test on. Actual nulls you don't want to display should be converted to blank strings in the code before this part ran. Or, edit the __hasValue function to do your own test.

Example:
$input = "Complex if: {=if value8+!value7+value9=something}Expression passed{=end if}";

%keys = (
	value1 => "Value of value 1",
	value2 => "0", 
	value3 => 0, 
	value4 => 9, 
	value5 => "0 but true", 
	value6 => null, 
	value7 => '', 
	value8 => 'null', 
	value9 => "something",
);

$response = stl_parser::parse_stl($input,\%keys);

print "REPONSE: " . $response . " END RESPONSE";
exit;
