# TAX Database
The idea is to build a machine readable database with all applicable taxes, based on the transaction.

## WHY?!?
- We all understand taxes, so why shouldn't machines?
- Right now every system that is able to create invoices has to manage their own dataset of countries, taxes and which one to choose for which country. When your bicicle web-shop starts shipping to the UK, you'll add it manually. Once the legal obligations change, everyone has to change it manually.
- We, especially the software business, live in a global market. When I want to sell my imaginary software to mexico I have to research all tax details by myself. Why don't we collaborate on this together?

## Usage
When there is all this data, how should it be used? Actually I don't care. You can do whatever you want with it. I for myself might create some open source project which would let you do something like this:

	// first parameter is the directory of this repository
	// second parameter is the country my company is registered in
	$taxDatabase = new TaxDatabase( '/some/directory/with/this/repository', 'de' );

	// now I have a visitor from the UK and I want to display the correct price, including tax:
	$tax = $taxDatabase->taxForCustomerFromCountry( 'uk', 'software' );
	
	// next I have a registered company from the netherlands that wants to buy my product
	$tax = $taxDatabase->taxForBusinessFromCountry( 'nl', 'software' );

This library is not part of this project, and will never be. This data is not bound to any language, library or framework. Everything that can parse JSON data can use it. [Open Data](http://en.wikipedia.org/wiki/Open_data) is about »the idea that certain data should be freely available to everyone to use and republish as they wish«. This is what these JSON files might be about – one day.

## Disclaimer
This data is not correct right now. It is your responsibility to check it according to your law. This repository is only a proof of concept and some groundwork to agree on some common format. 

If this format is stupid, you have a better one, or there is already an existing one, I would be glad to use that instead. I don't want to run this project, but someone has to start something. 

## Contribute
**Yes, please** do. Just [create a new issue](https://github.com/tobiastom/tax-database/issues/new) to start a discussion. About whatever you like. It would be awesome if it is somehow related to this project. **Thank you**.

## Format
### Countries
Each country is defined only once inside the `countries.json` file. Inside it there is human readable name and all the different taxes for that particular country.
The names for the taxes can be any string. They are only used inside the tax definition files.

A sample country record looks like this (as found in `countries.json`):

	"de": {
		"name": "Germany",
		"taxes": {
			"reduced": 7,
			"normal": 19
		}
	},

### Requirements
Some taxes come with requirements. The `requirements.json` can define these. An example for a requirement might be that there is no tax required at all if an invoice is send from a registered company to a registered company inside the EU, if both companies have a [VAT number](http://en.wikipedia.org/wiki/VAT_identification_number).

This file is not supposed to be understood by machines. If there are requirements for a tax, the software that uses this data has to handle it.

Sample requirements record (as found in `requirements.json`):

	"vat-number": {
		"description": "A value added tax identification number is required on invoices.",
		"more": "http://en.wikipedia.org/wiki/VAT_identification_number"
	}

### Tax definition
Tax definitions are based on categories. Not sure how to use them right now. I am only interested into the software business, that's why there is only a folder `software`. Others, like `books`, `ebooks`, `flowers` might be added later.
Each category has its own directory. Inside this directory there is a file for each country. The country file represents the location where the company, that will send an invoice, is located. If your company is registered in germany you would have to look into `de.json`.

The file is separated into two categories:

- **`business-to-client`** – Your company sends an invoice to an individual.
- **`business-to-business`** – Your company send an invoice to another company.

Each section has three properties:

- **`country`** – The country inside which the buyer is located.
- **`tax`** – The name of the tax that should be used. This is the exact same string as the tax definition inside the `country.json`.
- **`requirement`** – Requirements that have to be fulfilled that this tax is applicable.

When we combine the requirements and the country definition from above, a sample record for invoices from germany to germany would look like this:

	{
		"business-to-client": [
			{
				"country": [ "de" ],
				"tax": "normal",
				"requirement": []
			}
		],
		"business-to-business": [
			{
				"country": [ "de" ],
				"tax": null,
				"requirement": [
					"vat-number"
				]
			}
		]
	}
