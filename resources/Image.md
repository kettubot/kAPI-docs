# Images

Images are a custom structure that allow simple control over Kettu's social features.

## Image Structure

| field    | type    | description                                            |
| -------- | ------- | ------------------------------------------------------ |
| id       | integer | image id, unique to the category                       |
| category | string  | image category                                         |
| direct   | string  | direct URL to the image                                |
| source   | string  | URL to the image source page (prefer FA, DA over e926) |
| artist   | string  | URL to the artist's page                               |
| notes    | string  | notes for the image                                    |
| tags     | array   | list of image tags used for caption matching           |

### Image Categories

A list of image categories can be found using the List Categories endpoint.

## List Categories % GET /images % REQUIRES `READ_IMAGES`

Returns a list of image categories.

## List Category Images % GET /images/{image.category#docs:resources:image/image-structure} % REQUIRES `READ_IMAGES`

Returns a list of [image](#docs:resources:image/image-structure) objects within the specified category.

## Get Image % GET /images/{image.category#docs:resources:image/image-structure}/{image.id#docs:resources:image/image-structure} % REQUIRES `READ_IMAGES`

Returns a specific [image](#docs:resources:image/image-structure).

## Get Random Image % GET /images/{image.category#docs:resources:image/image-structure}/random % REQUIRES `READ_IMAGES`

Returns a random [image](#docs:resources:image/image-structure) in the given category. Will avoid sending duplicates with a history of 3 items.

## Create Image % POST /images/{image.category#docs:resources:image/image-structure} % REQUIRES `MANAGE_IMAGES`

Creates a new image in the given category.

pause