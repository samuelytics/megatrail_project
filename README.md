The Megatrail project is my attempt to detail the largest connected component of a network consisting of bike/hiking trails, sidewalks, and designated pedestrian crossings in a given region of the world, which I dub a "megatrail" for the purposes of this project. In this script, I use Chicago as an example. 

This currently consists of 3 parts.

# Gathering
The map-gathering script uses the overpass API for OpenStreetMap to gather all relevant nodes and ways (ie, nodes and ways that correspond to the appropriate pedestrian/cyclist infrastructure) from a given area. It then assembles a networkx graph from all of the gathered nodes and extracts the largest connected component (LCC) from them. It then uses a breadth-first search algorithm to extend itself; namely, it looks for cardinal directions in which the ways overextend the bounding box that was used to extract the initial nodes and ways, creates a bounding box of equal size in that direction, and adds it to a queue. It then takes the oldest member of a queue, extracts new relevant nodes and ways from its associated bounding box, adds it to the networkX graph, and creates a new LCC from this data. The process repeats until a designated cutoff point, or until there are no usable bboxes left (that is to say, the megatrail has been fully mapped).

To perform this yourself, use the Modular Megatrail Scrape notebook and run all cells until the segment labeled "Begin." Here, you'll need to provide your own user agent for the Overpass API, ideally in the format "\[Project Name\], (\[Your email address\])." You can also set the latitude and longitude for where you will want the lower left corner of your first bbox (by default, it starts in the Chicagoland area, roughly around Aurora. Make sure that whatever bbox you're using has sidewalks in it, otherwise the script will stop immediately.  

scrape_the_megatrail, by default, will require you to input the api, the latitude, and the longitude. You can alter the size of the bbox using the interval parameter (I find that 0.1 tends to not overload the API), as well as how long you want to continue the process using the box_limit parameter (the Chicago megatrail requires around 89 bboxes or so, but this process takes a while, so the default is 40). You can check the results converting lcc_ways to borders using convert_to_dispersed_borders, then running quick_map on the results. If there are insufficient boxes, you can re-run scrape_the_megatrail, this time including the used_bboxes, total_result_nodes, total_result_ways, and save_bbox_dict that you got from the initial run, labeled under their respective parameters (ie, used_bboxes is the "used_bboxes" parameter in the script). 

You can then save the results using save_the_pickles, making sure to include the appropriate prefix for your filenames. 

# Community Analysis

I'm currently performing community analysis on the Chicago megatrail, in an effort to better grasp its overall structure. I initially used the Louvain algorithm, but found that the communities it formed were too granular to be useful. Geography-based k-means clustering did not pay attention to the connectivity of the nodes, and node2vec-based k-means clustering was too computationally intensive. As such, I am using Clauset-Newman-Moore greedy modularity maximization, as provided by the networkx library's greedy_modularity_community function. Currently 7 analyses are available on the website, consisting of 8, 10, 12, 14, 16, 18, and 20 communities, using the cutoff function on greedy_modularity_community's best_n and cutoff parameters. A previous version of the analysis was made using varying levels of the resolution parameter; analyzing the difference between the two is one of the future goals of this project. 


For each collection of communities, I have measured the following metrics: 
1. Total Distance: The kilometers of bike trail and sidewalk present within the community.
2. Convex Hull Area of Region: The area, in square kilometers, of a the smallest convex polygon to encompass the community.
3. Intersections per Distance: The number of intersections (ie, nodes with degrees of 3 or more), per kilometer of bike trail/sidewalk within the community.
4. Kilometers per Convex Hull Area: The number of kilometers of sidewalk/trail per square kilometer of convex hull area (see #2).
5. Bridge Intersections per Convex Hull Area: How many intersections serve as [bridges](https://en.wikipedia.org/wiki/Glossary_of_graph_theory#bridge) between two parts of the community (that is to say, the number of intersections which, if removed, would split the community into 2 pieces) that exist per square kilometer of the community.
6. Bridge Intersections per Distance: How many intersections serve as bridges per kilometer of trail/sidewalk within the community.

To perform this process and save the resulting metrics and geojsons that are necessary for visualization, take the saved outputs outputs from the scraping process and load them using the load_pickles command in Modular_Megatrail_Analysis (the second cell; you'll need to define the prefix and directory you're using). Then scroll to "output measures" and run the run_greedy_modularity cell. You can alter the resolution of run_greedy_modularity using the resolution parameter; by default it's .0001. To visualize the layers of the map, run the map_those_layers cell. To get the analysis, run the run_analysis cell. Finally run the save_files cell to get the .csv and geojson needed for visualization. 

# Visualization

I have created a [website](https://sorinash.neocities.org/) for visualizing both maps of the communities along with bar graphs of their metrics (both for the 7 and 12-community collections), using a combination of Leaflet for geojson visualization and d3 for data visualization. 

# Preliminary Findings

## Finding communities at varying resolutions alters the shape and distribution of communities when compared to finding communities at varying cutoff points. 

Broadly speaking, using small resolutions to find small numbers of communities creates larger, "blockier" communities than using defined cutoff points. Using a resolution of 1 and defining the cutoff points to define the number of communities results in a number of communities radiating outwards from the city of Chicago, whereas using smaller resolutions and no defined cutoff points to create 7 and 12 communities divided Chicago into, at most, two communities. Further analysis will be performed to determine distinctions between these methods (both in terms of qualitative observation, quantitative analysis, and overall utility). 

## Communities within the City of Chicago have the densest amount of pedestrian/cyclist infrastructure

At all levels of community subdivision, the communities with the highest number of kilometers of trail per square kilometer of convex hull area are predominately located within the City of Chicago. This is to be somewhat expected, as dense urban areas are generally less car-dependent than suburban or rural regions. 

## Suburban communities tend to be defined by bike trails

In the western suburbs, the Fox River Trail provides a visible backbone for multiple communities, whereas within the northeastern suburbs, a community rapidly coalesces around the Des Plaines River Trail. I'm currently determining a quantitative metric for this observation. If true, this may be due to the tendency of bike trails to provide easy connections between municipalities; for instance, the Fox River Trail crosses multiple highways that are not crossed by sidewalks. 



# Current efforts

1. Compare community-size- and resolution- based methods of partitioning in terms of utility and metrics.
2. Determine consistency of nodes as a measure of distance (ie, how certain can we be that there are X number of nodes in a mile; this can help with additional metric down the line). 
3. Reconfigure metrics, particularly regarding bridge intersections.
4. Consider city- or town-based metrics as well, such as "proportion of town that's in a community" or "number of towns in a community." Could extend to neighborhood-based 
5. Work with OSM community to get better mapping of area sidewalks and bike trails.
6. Try pipeline on other regions of the United States. 
