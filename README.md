Package validator
================

Package validator implements variable validations

Fork of https://github.com/go-validator/validator with additional buildin validators.

Installation
============

Just use go get.

	go get github.com/THE108/validator

And then just import the package into your own code.

	import (
		"github.com/THE108/validator"
	)

Usage
=====

Please see http://godoc.org/gopkg.in/validator.v2 for detailed usage docs.
A simple example would be.

	type NewUserRequest struct {
		Username string `validate:"min=3,max=40,regexp=^[a-zA-Z]$"`
		Name string     `validate:"nonzero"`
		Age int         `validate:"min=21"`
		Password string `validate:"min=8"`
	}

	nur := NewUserRequest{Username: "something", Age: 20}
	if err := validator.Validate(nur); err != nil {
		// values not valid, deal with error here
	}


Builtin validators

Here is the list of validators buildin in the package.

	len
		For numeric numbers, max will simply make sure that the
		value is equal to the parameter given. For strings, it
		checks that the string length is exactly that number of
		characters. For slices,	arrays, and maps, validates the
		number of items. (Usage: len=10)
	
	max
		For numeric numbers, max will simply make sure that the
		value is lesser or equal to the parameter given. For strings,
		it checks that the string length is at most that number of
		characters. For slices,	arrays, and maps, validates the
		number of items. (Usage: max=10)
	
	min
		For numeric numbers, min will simply make sure that the value
		is greater or equal to the parameter given. For strings, it
		checks that the string length is at least that number of
		characters. For slices, arrays, and maps, validates the
		number of items. (Usage: min=10)
	
	nonzero
		This validates that the value is not zero. The appropriate
		zero value is given by the Go spec (e.g. for int it's 0, for
		string it's "", for pointers is nil, etc.) Usage: nonzero
	
	regexp
		Only valid for string types, it will validate that the
		value matches the regular expression provided as parameter.
		(Usage: regexp=^a.*b$)

	(in this fork)

	in
		Validates that the value is in enum. Only valid for string types.
		(Usage: in=ios|android|windows)

	ipv4
		Validates that the value is valid ipv4. Only valid for string types.
		Usage: ipv4.

	url
		Validates that the value is valid url. Only valid for string types. 
		Possible schemas: ftp, http, https. Usage: url.

	path_valid
		Validates that the value is valid unix path. Only valid for string types.
		Usage: path_valid.

	path_exists
		Validates that the value is valid unix path and that path exists.
		Only valid for string types. Usage: path_exists.

	basepath_exists
		Validates that the value is valid unix path and that path exists.
		Only valid for string types. Usage: path_exists.

Custom validators

It is possible to define custom validators by using SetValidationFunc.
First, one needs to create a validation function.

	// Very simple validator
	func notZZ(v interface{}, param string) error {
		st := reflect.ValueOf(v)
		if st.Kind() != reflect.String {
			return errors.New("notZZ only validates strings")
		}
		if st.String() == "ZZ" {
			return errors.New("value cannot be ZZ")
		}
		return nil
	}

Then one needs to add it to the list of validators and give it a "tag"
name.

	validator.SetValidationFunc("notzz", notZZ)

Then it is possible to use the notzz validation tag. This will print
"Field A error: value cannot be ZZ"

	type T struct {
		A string  `validate:"nonzero,notzz"`
	}
	t := T{"ZZ"}
	if valid, errs := validator.Validate(t); !valid {
		fmt.Printf("Field A error: %s\n", errs["A"][0])
	}

You can also have multiple sets of validator rules with SetTag().

	type T struct {
		A int `foo:"nonzero" bar:"min=10"`
	}
	t := T{5}
	SetTag("foo")
	validator.Validate(t) // valid as it's nonzero
	SetTag("bar")
	validator.Validate(t) // invalid as it's less than 10

SetTag is probably better used with multiple validators.

	fooValidator := validator.NewValidator()
	fooValidator.SetTag("foo")
	barValidator := validator.NewValidator()
	barValidator.SetTag("bar")
	fooValidator.Validate(t)
	barValidator.Validate(t)

This keeps the default validator's tag clean. Again, please refer to
godocs for a lot of more examples and different uses.

Pull requests policy
====================

tl;dr. Contributions are welcome.

The repository is organized in version branches. Pull requests to, say, the
`v2` branch that break API compatibility will not be accepted. It is okay to
break the API in master, *not in the branches*.

As for validation functions, the preference is to keep the main code simple
and add most new functions to the validator-contrib repository.

https://github.com/go-validator/validator-contrib

For improvements and/or fixes to the builtin validation functions, please
make sure the behaviour will not break existing functionality in the branches.
If you see a case where the functionality of the builtin will change
significantly, please send a pull request against `master`. We can discuss then
whether the changes should be incorporated in the version branches as well.

License
=======

Copyright 2014 Roberto Teixeira <robteix@robteix.com>

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
