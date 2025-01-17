<a href="https://www.sparkpost.com"><img src="https://www.sparkpost.com/sites/default/files/attachments/SparkPost_Logo_2-Color_Gray-Orange_RGB.svg" width="200px"/></a>

[Sign up](https://app.sparkpost.com/sign-up?src=Dev-Website&sfdcid=70160000000pqBb) for a SparkPost account and visit our [Developer Hub](https://developers.sparkpost.com) for even more content.

# SparkPost API Documentation

[![Run in Postman](https://s3.amazonaws.com/postman-static/run-button.png)](https://www.getpostman.com/run-collection/81ee1dd2790d7952b76a)

## Prerequisites

* You have installed [Git](http://git-scm.com/downloads) on your development machine
* You have installed [Node.js and NPM](https://nodejs.org/) on your development machine

The SparkPost API Docs in Apiary Blueprint format (based on Markdown)

## Installation

1. Clone the repository

```git clone https://github.com/SparkPost/sparkpost-api-documentation```

2. Change into the directory

2. We use [Grunt](http://gruntjs.com/) for our task runner, so you will also have to install Grunt globally

```npm install -g grunt-cli```

3. Install the dependencies

```npm install```

## Development

### Grunt Commands

#### Apiary Blueprint Validator

Once all the dependencies are installed, you can execute the Apiary Blueprint Validator tests in the following ways:

* Run the test on ALL /services/ files sequentially
  ```grunt testFiles```

* Run the test an individual /services/ file
  ```grunt shell:test:<filename>```

#### Compile and Test

* Concatenate ALL /services/ files into a single apiary.apib file, then test the compiled file
  ```grunt compile```

### Contributing
[Guidelines for adding issues](docs/ADDING_ISSUES.markdown)

[Apiary Blueprint Specification](https://github.com/apiaryio/api-blueprint/blob/master/API%20Blueprint%20Specification.md)

[Submitting pull requests](docs/CONTRIBUTING.markdown)

## License

Read the [complete license](/LICENSE) information for our Sparkpost API documentation.
