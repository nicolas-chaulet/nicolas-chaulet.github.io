+++
author = "Nicolas Chaulet"
title = "Exploring the largest public EPD dataset for construction materials"
date = "2026-01-12"
description = "This post takes a deep dive into the content of the EC3 dataset and shows how the dataset can be exposed in a nice and performant way."
+++

I like to learn by exploring and building. And I wanted to learn more about Environmental Product Declaration (EPDs) and take a deeper dive into the online [EC3 database](https://www.buildingtransparency.org/tools/ec3/). This database is the largest repository of EPDs for a range of construction materials. As I started using EC3 I quickly got a bit frustrated by the lack reactivity of the search experience, I get it, this tool is not meant for speed and performance but it started to wear on me a little. So I decided to build myself a tool that would allow me to explore the dataset easily. At the same time I wanted to play with some tools I wasn’t familiar with and get a little refresher about what’s out there in terms of front-end development and data science.

Here is what I ended up with:

{{< video src="images/ec3-demo.mp4" >}}

## The plan

The idea was simple, gather the digitized EPDs hosted by EC3, clean them up, look at what sort of information they hold and expose the dataset via a nice super responsive UI that would allow me to search and filter the data.

## Getting the data

Building Transparency (the organization that hosts EC3) provides a couple of API that allows users to download the data they host in a structured format. One of those APIs is supposed to return data in the [OpenEPD](https://www.open-epd-forum.org/) format (https://openepd.buildingtransparency.org/api) but I found that it is very unstable and fails fairly often. Instead, I favored using the API documented as part of the EC3 database (https://buildingtransparency.org/api/), it gives more data that is harder to reason about but it is a bit more robust. You still need to have some retry logic in place and be ready to stop the download and retry later since sometimes the API throws errors for a little while. Just as a note, both APIs need an API key.

With a little bit of care and patience I managed to get the 204,955 digitized EPDs onto my local computer, each digitized EPD is made of 472 datapoints. I did not want to do this too often, so the first thing I did after having downloaded the data was to group it into a single [Polars](https://pola.rs/) data frame and save it to disk for faster access later on. For those of you that are not familiar with the Python data science ecosystem, Polars is a Python package for data science, it allows you to do all kinds of data manipulation (aggregation, filtering, cleanup) very easily and very quickly.

## Some data exploration and cleanup

After looking at the data for a bit, a few things stood out

- of the 472 columns available for each EPD many are actually empty, we just removed those,
- some EPDs are not valid and are marked as such by EC3, we will remove those from the dataset (more than 15,000 of them at the time of writing),
- finally, many columns have a value for just a few products, the reason is that most of those columns are very specific to a certain type of material and are only relevant for a handful of them (the attribute `membrane_roofing_reinforcement` for example is only defined for 4 EPDs).

After this basic cleanup I also created a simpler version of the dataset that only retains information that I wanted to search or filter by:

- the EPD geographical applicability,
- global Warming Potential (GWP) as a a number in kgCO2eq,
- the product’s name, description and picture when they have one,
- the product’s main category and sub category,
- the date the digitized EPD was created and the data it was updated last,
- and a link to the original EC3 page.

## The search interface

Below are some of the things I really want to do as part of this work

- sub 100ms latency for any kind of query
- ability to search using a text input or filter for certain categories
- pagination, I want to be able to know how many products a certain search would yield and go through pages of 20 results,
- reactive categorical filtering, I want to be able to know what sort of filter make sense at a given moment. For example if I filter for “Concrete” as a main category, I should not be able to also filter by “Chairs” as a subcategory since this combination would yield no results. This is called faceting in classic search terminology
- finally, I settled for a tabular UI since the data is not very visual, many EPDs in EC3 don’t have a meaningful image associated with it.

Now in terms of the actual implementation, since I started looking into Polars I decided to stick to it as a backend, it is actually very efficient and allowed me to get a search backend setup in no time. The functionality is exposed to the frontend via [FastAPI](https://fastapi.tiangolo.com/) server. The frontend is a react application and I went quite deep into the set of libraries provided by [Tanstack](https://tanstack.com/), I was familiar with some of their work but wanted to test out some the more recent packages. All in all, it went quite smoothly and I am quite happy with the result.

I did some performance checking and the current stack can happily handle 10 requests per second when running it on a single thread which is quite nice a starting point and plenty for my own use.

## Next steps

This is obviously a very basic setup and would most likely not scale very well. One of the things I would like to explore is to replace the search with a proper tool for search such as [Typesense](https://typesense.org/). As a side note, they are great! I would expect such a switch to allow the search to still perform at similar speeds but it would be more scalable in terms of concurrency, more users could sue the tool at the same time. Another nice side effect is that Typesense supports semantic search out of the box enabling fuzzy matching which is actually fairly important when looking for an EPD. Finding similar products is another thing that would become fairly easy to implement.

## References

- Polars documentation https://docs.pola.rs/api/python/stable/reference/index.html
- Building Transparency and EC3 https://www.buildingtransparency.org/tools/ec3/
- What is an EPD https://carbonleadershipforum.org/environmental-product-declarations-epd-101/
- General reading about embodied carbon for the build environment [https://carbonleadershipforum.org](https://carbonleadershipforum.org/)
