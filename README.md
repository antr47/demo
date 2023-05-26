Migrations can be used to manage the contents of Directus collections (e.g. initial hydration). In order to do it, you must ensure that the schema is up to date before running your migrations.
# Installation
`npm i directus-migration-tools`
#  Usage
All migrations must be located in the `extensions/migrations` folder.

## File Name 
The file name follows the following structure: `[identifier]-[name].js` for example: `20201202A-my-custom-migration.js`

## Config collection schema
#### Create collection config:
Example file name: CDH20230425A-create.js
```
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
            meta: {
                color: "#B31B1B",
                group: "projects",
                display_template: "{{translations}}",
            },
        },
        fields: {
            ...defaultFields,
            id: generateField.genPrimaryKey(),
            translations: generateSpecField.translations("languages", "projects_categories_translations",
                {
                    title: generateField.genNormal(),
                },
                {
                    meta: {
                        display: "translations",
                        display_options: {
                            template: "{{title}}",
                            languageField: "name",
                            defaultLanguage: "vi-Vi",
                            userLanguage: true,
                        },
                    },
                }
            ),
        },
    },
    {
        collection: {
            name: "services_items",
            meta: {
                color: "#B31B1B",
                group: "services",
                display_template: "{{translations}}",
                hidden: true,
            },
        },
        fields: {
            ...defaultFields,
            id: generateField.genPrimaryKey(),
            icon: generateField.genNormal(),
            translations: generateSpecField.translations("languages", "services_items_translations",
                {
                    title: generateField.genNormal(),
                    description: generateSpecField.textArea(),
                },
                {
                    meta: {
                        display: "translations",
                        display_options: {
                            template: "{{title}}",
                            languageField: "name",
                            defaultLanguage: "vi-Vi",
                            userLanguage: true,
                        },
                    },
                }
            ),
        },
    },
    {
        collection: {
            name: "projects_contents",
            meta: {
                color: "#B31B1B",
                group: "projects",
                display_template: "{{subtitle}}",
            },
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
                {
                    text: "Solutions",
                    value: "2",
                },
                {
                    text: "Related Projects",
                    value: "3",
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
            meta: {
                color: "#B31B1B",
                group: "services",
                display_template: "{{subtitle}}",
            },
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
                {
                    text: "Experiences",
                    value: "2",
                },
                {
                    text: "Faqs",
                    value: "3",
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
### Update collection config:
Example file name:  CDH20230425B-update.js
```
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
            items: generateField.generateM2m("services_items","services_items_related", {
                meta: {
                    width: "half",
                },
            }),
            projects: generateField.generateM2m("projects", "services_projects_related", {
                meta: {
                    width: "half",
                },
            }),
            tags: generateField.generateM2m("tags","services_tags", {
                meta: {
                    width: "half",
                },
            }),
            faqs: generateField.generateM2m("faqs","services_faqs", {
                meta: {
                    width: "half",
                },
            }),
        },
    },
    {
        collection: {
            name: "services_translations",
        },
        fields: {
            subtitle: generateField.genNormal("string"),
            subdescription: generateSpecField.textArea(),
            statistics: generateSpecField.repeater({
                meta: {
                    options: {
                        fields: [
                            {
                                field: "quantity",
                                name: "quantity",
                                type: "string",
                                meta: {
                                    field: "quantity",
                                    type: "string",
                                    interface: "input",
                                },
                            },
                            {
                                field: "description",
                                name: "description",
                                type: "string",
                                meta: {
                                    field: "description",
                                    type: "string",
                                    interface: "input",
                                },
                            },
                        ],
                    },
                },
            }),
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
            solutions: generateField.generateM2m("solutions", "projects_solutions_related", {
                meta: {
                    width: "half",
                },
            }),
            related_projects: generateField.generateM2m("projects", "projects_projects_related", {
                meta: {
                    width: "half",
                },
            }),
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


    
