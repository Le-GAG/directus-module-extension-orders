# Le GAG - Directus orders extension

[Directus](directus) business-logic module to display an overview of orders.  
Highly coupled to [Le GAG](https://github.com/Le-GAG)'s website.


## Usage

Because of a [bug in Parcel](parcel-bug), a nasty workaround is needed:

````shell script
npm install
sed -i.bak '/publicUrl/ a\
\ \ \ \ sourceMaps: false,
' node_modules/@directus/extension-toolkit/cli/cmds/{build,watch}.js
````

### Build for production

````shell script
npm run build --input src
````

### Build for development

````shell script
npm run dev --input src --output ../api/public/extensions/custom/modules/orders
````


## Installation

Create a production build (`npm run build`) and copy the content of the _dist_ 
directory to the _public/extensions/custom/modules/le-gag-orders_ directory of a 
[Directus API](directus-api) instance.

[directus]:     https://directus.io/
[directus-api]: https://github.com/directus/api
[parcel-bug]:   https://github.com/parcel-bundler/parcel/issues/2795
