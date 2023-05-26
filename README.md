## Installation

`npm i directus-migration-tools`

## Usage

### Schema Config

The file name follows the following structure: `[identifier]-[name].js` for example: `20201202A-my-custom-migration.js`

> **Note**: All migration files must be located in the Directus `extensions/migrations` folder.

### Create Related Fields:

-   Many to one: `generateM2o: (related_collection,options)`

Example:

```javascript
   projects_translation: generateField.generateM2o(
                "projects_translations",
                {
                    field_o2m: {
                        create: true,
                        field_name: "contents",
                    },
                    meta: {
                        hidden: true,
                    },
                }
            ),
```

-   Many to many: `generateM2m: (related_collection,temp_collection, options , fields_extend)`

Example:

```javascript
  projects: generateField.generateM2m("projects", "services_projects_related"),
```

-   One to many: `generateO2m: (related_collection,related_field ,options)`

Example:

```javascript
menu_items: generateField.generateO2m("menu_item", {
    related_field: "menu",
    meta: {
        sort: 6,
    },
});
```

### Create Config Example:

-   File name: `CDH20230425A-create.js`

```javascript
const {
    generateField,
    generateSpecField,
    upCreateKnex,
    downCreateKnex,
} = require("directus-migration-tools");

const defaultFields = {
    user_created: generateSpecField.userCreated(),
    date_created: generateSpecField.dateCreated(),
    user_updated: generateSpecField.userUpdated(),
    date_updated: generateSpecField.dateUpdated(),
    status: generateSpecField.status(),
};

const config = [
    {
        collection: {
            name: "projects_categories",
        },
        fields: {
            ...defaultFields,
            id: generateField.genPrimaryKey(),
            translations: generateSpecField.translations(
                "languages",
                "projects_categories_translations",
                {
                    title: generateField.genNormal(),
                }
            ),
        },
    },
    {
        collection: {
            name: "services_items",
        },
        fields: {
            ...defaultFields,
            id: generateField.genPrimaryKey(),
            icon: generateField.genNormal(),
            translations: generateSpecField.translations(
                "languages",
                "services_items_translations",
                {
                    title: generateField.genNormal(),
                    description: generateSpecField.textArea(),
                }
            ),
        },
    },
    {
        collection: {
            name: "projects_contents",
        },
        fields: {
            ...defaultFields,
            id: generateField.genPrimaryKey(),
            module: generateSpecField.radioButton([
                {
                    text: "Overview",
                    value: "0",
                },
                {
                    text: "Problems",
                    value: "1",
                },
            ]),
            title: generateField.genNormal("string"),
            subtitle: generateField.genNormal("string"),
            description: generateSpecField.textArea(),
            sort: generateSpecField.sort({
                meta: {
                    hidden: false,
                },
            }),
            images: generateSpecField.files("projects_contents_files"),
            content: generateSpecField.wysiwyg(),
            projects_translation: generateField.generateM2o(
                "projects_translations",
                {
                    field_o2m: {
                        create: true,
                        field_name: "contents",
                    },
                    meta: {
                        hidden: true,
                    },
                }
            ),
        },
    },
    {
        collection: {
            name: "services_contents",
        },
        fields: {
            ...defaultFields,
            id: generateField.genPrimaryKey(),
            module: generateSpecField.radioButton([
                {
                    text: "Overview",
                    value: "0",
                },
                {
                    text: "Services",
                    value: "1",
                },
            ]),
            title: generateField.genNormal("string"),
            subtitle: generateField.genNormal("string"),
            description: generateSpecField.textArea(),
            sort: generateSpecField.sort(),

            position: generateSpecField.radioButton(
                [
                    {
                        text: "Left",
                        value: "0",
                    },
                    {
                        text: "Right",
                        value: "1",
                    },
                ],
                {
                    meta: {
                        hidden: true,
                        conditions: [overviewCondition],
                    },
                }
            ),
            thumbnail: generateSpecField.image(),
            content: generateSpecField.wysiwyg(),
            services_translation: generateField.generateM2o(
                "services_translations",
                {
                    field_o2m: {
                        create: true,
                        field_name: "contents",
                    },
                    meta: {
                        hidden: true,
                    },
                }
            ),
        },
    },
];

module.exports = {
    async up(knex) {
        await upCreateKnex(knex, config);
    },
    async down(knex) {
        await downCreateKnex(knex, config);
    },
};
```

### Update Config Example:

-   File name: `CDH20230425B-update.js`

```javascript
const {
    generateField,
    generateSpecField,
    upUpdateKnex,
    downUpdateKnex,
} = require("directus-migration-tools");

const config = [
    {
        collection: {
            name: "services",
        },
        fields: {
            items: generateField.generateM2m(
                "services_items",
                "services_items_related"
            ),
            projects: generateField.generateM2m(
                "projects",
                "services_projects_related"
            ),
            tags: generateField.generateM2m("tags", "services_tags"),
            faqs: generateField.generateM2m("faqs", "services_faqs"),
        },
    },
    {
        collection: {
            name: "services_translations",
        },
        fields: {
            subtitle: generateField.genNormal("string"),
            subdescription: generateSpecField.textArea(),
        },
    },
    {
        collection: {
            name: "projects",
            meta: {
                display_template: "{{translations}}",
            },
        },
        fields: {
            banner: generateSpecField.image(),
            website: generateField.genNormal(),
            published_date: generateSpecField.dateTime(),
            color: generateField.genNormal("string", {
                meta: {
                    interface: "select-color",
                },
            }),
            partner: generateField.generateM2o("partners"),
            category: generateField.generateM2o("projects_categories"),
            solutions: generateField.generateM2m(
                "solutions",
                "projects_solutions_related"
            ),
            related_projects: generateField.generateM2m(
                "projects",
                "projects_projects_related"
            ),
        },
    },
];

module.exports = {
    async up(knex) {
        await upUpdateKnex(knex, config);
    },
    async down(knex) {
        await downUpdateKnex(knex, config);
    },
};
```

#### Note:

> Migrations have to export an `up` and a `down` function. These functions get a [Knex](http://knexjs.org/) instance that can be used to do virtually whatever.
