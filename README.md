# Request for Comments on proposed BigCommerce Cart API

Welcome to the BigCommerce Cart API RFC!

In this document, we have opened up our proposed Cart API schema to solicit the developer community's feedback. Our goal is to ensure you, as a BigCommerce partner or developer, have the tools you need to build robust tools & cohesive integrations that benefit our merchants (your clients) and push our platform and ecosystem forward.

Building a world-class API is an iterative task. Our first release of this Cart API will be read-only, allowing you to access the contents & details of a BigCommerce cart on the storefront (via JavaScript) and also via our backend HTTP REST API. Future releases will enable writable access to parts of the cart, unblocking many more possibilities and use cases.

Once we have gathered feedback, we will take time to digest all of the community's feedback and improve the API. The end result will be published on [BigEng.io](http://bigeng.io) once we have finalized the API specifications. This will detail the feedback that has been incorporated (or not) and context for those changes.

Our initial specification takes into consideration use cases such as:
- Retrieve an abandoned cart
- Reconstruct a cart from a URL
- Support multiple shipping destinations for a single cart quote
- Collect cart analytics and build actionable reports
- [your cool idea here]

This should help empower email remarketing, cart upsells, cross-browser carts for the same customer, and more.

## Introduction to Swagger API Documentation

[Swagger](http://swagger.io/) serves as a set of rules for a format describing REST APIs. The format is both machine-readable and human-readable. Descriptions of each field are included inline to help disambiguate terms (we might call things by different names internally than what you're used to).

If you are looking for something more readable, we suggest trying out the [Swagger browser editor](https://editor.swagger.io/#/?import=https://raw.githubusercontent.com/bigcommerce/api/master/swagger/checkout-draft.yaml).

## How to provide feedback

To provide your feedback for how we might improve this API, please submit comments on the latest pull request - commenting directly on lines ([instructions here](https://developer.github.com/guides/working-with-comments/#pull-request-comments-on-a-line)) will help target your feedback to particular areas. This is preferred for streamlined conversation and centralizing the discussion.

