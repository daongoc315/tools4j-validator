[![Build Status](https://travis-ci.org/relvaner/tools4j-validator.svg?branch=master)](https://travis-ci.org/relvaner/tools4j-validator)
[![Coverage Status](https://coveralls.io/repos/github/relvaner/tools4j-validator/badge.svg?branch=master)](https://coveralls.io/github/relvaner/tools4j-validator?branch=master)
[Note: Coverage under Windows 85,7%]

Why another validation framework?
=================================

Validation frameworks are often conceptualized for Java Beans. Motivation for this project was, that there is no simple solution in cycle, for validating individual data types of Java Swing applications. This framework is also particularly useful, if it can not be determined a priori, which types of data should be validated (dynamic validation in combination with validation properties in JSON format). It is especially useful when you want to define validation constraints separated from business logic. This can be the case for database data validation, where validation requirements can dynamically change.


Introduction
============

The framework is used for checking the data for correctness. Specifically data can be checked for correctness by means of regular expression and the value range or field length for strings. Additionally own validators can be written, that can then define additional verifiable criteria (derived classes of type Validator &lt;T&gt;). Validation is a maximum of three steps (Transformation → Validation → BusinessRule). When passing a string, the string is first transformed to the desired data type. During the transformation process, decimal delimiters will transferred to a point notation (comma → point). Then, the actual validation (pattern, value range, length range) is performed. At the conclusion, rules can be checked for compliance. In Table 1 is a list of the various validators and its constraints to see.

Overview
========

| Validator | Constraints |
| --------- | ----------- |
| StringValidator     | NotNull, Null, NotEmpty, Empty, Pattern, Min, Max, Size |
| EmailValidator      | NotNull, Null, NotEmpty, Empty, Pattern, Min, Max, Size |
| PickListValidator   | NotNull, Null, NotEmpty, Empty, Pattern, Min, Max, Size, List |
| DateValidator       | NotNull, Null, Pattern, Valid, Past, Future |
| BigDecimalValidator | NotNull, Null, Min, Max, Range |
| BigIntegerValidator | NotNull, Null, Min, Max, Range |
| DoubleValidator     | NotNull, Null, Min, Max, Range |
| FloatValidator      | NotNull, Null, Min, Max, Range |
| LongValidator       | NotNull, Null, Min, Max, Range |
| IntegerValidator    | NotNull, Null, Min, Max, Range |
| ShortValidator      | NotNull, Null, Min, Max, Range |
| ByteValidator       | NotNull, Null, Min, Max, Range |
| BooleanValidator    | NotNull, Null, AssertFalse, AssertTrue |

Table 1: Overview of validators and possible constraints.

Examples
========
<b>Example of a StringValidator:</b>

Validation with a pattern, followed by the validation and an output of an error message and the type of an error message.
<pre><code>StringValidator v = new StringValidator();
v.setPattern("[A-Za-z]+");

v.validate("122");

System.out.println(v.getViolationMessage());

The input value "122" is not allowed ("[A-Za-z]+" expected)!

System.out.println(v.getViolationConstraint());

Pattern</code></pre>
<b>Example of an IntegerValidator:</b>

Validation with specifying a minimum, followed by the validation and an output of an error message.
<pre><code>IntegerValidator v = new IntegerValidator();
v.setMin(25);

v.validate(23);

System.out.println(v.getViolationMessage());

The input value "23" is out of range (>=25)!</code></pre>

Language Support for the output of error messages (currently German and English):

<pre><code>Validator.setViolationMessageLanguage("de");</code></pre>

Swing
=====

In Swing, text fields can be validated at runtime using the ValidationDocumentFilter class (see Fig. 1).

<pre><code>StringValidator v = new StringValidator();
v.setPattern("[A-Za-z]+");

TextField tf = new TextField("ABCD");
AbstractDocument doc = (AbstractDocument) tf.getDocument();
doc.setDocumentFilter(new ValidationDocumentFilter(window, v));</code></pre>

![alt text](https://github.com/relvaner/tools4j-validator/blob/master/doc/images/ValidationDocumentFilter.jpg "ValidationDocumentFilter")

Fig. 1: Validation of a text field with an error message shown.

As an alternative, use an InputVerifier (here: ValidationInputVerifier) instead:

<pre><code>tf.setInputVerifier(new ValidationInputVerifier(window, v)); // TextField</code></pre>

---

You can redirect the ViolationMessage also to a ValidationLabel (output):

<pre><code>doc.setDocumentFilter(new ValidationDocumentFilter(v, tf, output));</code></pre>

or

<pre><code>tf.setInputVerifier(new ValidationInputVerifier(v, output)); // TextField</code></pre>

Dynamic Validation
==================
A dynamic validation is made possible by coding the validation properties in JSON format. Definition of validation properties for an integer field in JSON:

<pre><code>{
    "type": "Integer",
    "constraints": [
        {"constraint": "NotNull"},
        {"constraint": "Range", "min": 2, "max": 10}
                   ]
}</code></pre>

<b>Example:</b>

Given are the constraints and the value which has to be checked. Using the ValidatorFactory, the type of validator is determined. Then the corresponding validator is instantiated and then the validation is performed.

<pre><code>String constraints = "{ … }";
String type = ValidatorFactory.parseType(constraints);

Validator&lt;?&gt; v = ValidatorFactory.createValidator(type);
v.setConstraints(constraints);

v.validateString(value);</code></pre>

Validation properties in JSON syntax obtained by:

<pre><code>v.getConstraintsAsJsonObject().toString();</code></pre>

Business Rules
==============

In the following, the use of business rules will be explained with an example. In this example, the passed date of birth should be checked on a minimum age of 18 years (age of majority). First, a business rule is written to verify the age of majority and added to the Business Rules Manager (see below).

<pre><code>BusinessRulesManager manager = new BusinessRulesManager();
String id = manager.addRule("AdultCheck", new BusinessRule<Calendar>() {
	@Override
	public boolean checkBusinessRule(Calendar value) {
		// AdultCheck
		Calendar now = Calendar.getInstance();
		now.set(Calendar.YEAR, now.get(Calendar.YEAR)-18);
				
		return value.compareTo(now)<=0;
	}
});</code></pre>

In the next step the apply of a business rule for the validator will be activated. This is done by adding the corresponding ID of the Business Rule and registering a handler for the business rule (here: DefaultBusinessRuleListener).

<pre><code>DateValidator v = new DateValidator();
v.setBusinessRuleID(id);
v.setBusinessRuleListener(new DefaultBusinessRuleListener(manager));
v.setPast(true);
v.setPattern("dd.MM.yyyy");
v.validateString("12.06.2004");

System.out.println(v.getViolationMessage());
System.out.println(v.getViolationConstraint());
</code></pre>

Error message:

<pre><code>The input value is not compliant to rules  ("AdultCheck")!</code></pre>