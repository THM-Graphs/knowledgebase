https://neo4j.com/blog/developer/claude-converses-neo4j-via-mcp/

Last week, Anthropic published a new [opens in new tabModel Context Protocol](https://www.anthropic.com/news/model-context-protocol) (MCP) for LLMs like Claude to communicate with external data sources.

This protocol allows, for instance, local applications like Claude.ai, Zed, Replit, and others to access file systems, APIs, and databases as part of the conversation with the user to fetch additional information. This can be used to answer questions, fetch more information, or use code generation to generate on-the-fly charts and visualizations.

Here’s what it could look like in your conversation.

![](https://cdn-images-1.medium.com/max/1024/1*UnC-f-LmG5RB-wk3G16TgQ.png)

Claude fetching data from Neo4j to render a heat map of ratings

If you want to test it yourself, you can add this config to your claude_desktop_config.json and restart your server to follow along. This points to a read-only demo database.

{"mcpServers": {  
      "movies-neo4j": {  
        "command": "uvx",  
        "args": ["mcp-neo4j-cypher",   
                 "--db-url", "neo4j+s://demo.neo4jlabs.com",   
                 "--user", "recommendations",   
                 "--password", "recommendations"]  
      }     
    }  
 }

And they didn’t just publish the protocol. They open-sourced a [opens in new tabnumber of connectors (servers)](https://github.com/modelcontextprotocol/servers) with an invitation to the community to add more.

## The Neo4j MCP Server(s)

We immediately started [opens in new tabimplementing a Neo4j MCP server](https://github.com/neo4j-contrib/mcp-neo4j/tree/main/servers/mcp-neo4j-cypher) to see what that would look like, which we also [opens in new tabpublished on PyPI](https://pypi.org/project/mcp-neo4j-cypher/), so you can run it directly.

[opens in new tabGitHub – neo4j-contrib/mcp-neo4j: Model Context Protocol with Neo4j](https://github.com/neo4j-contrib/mcp-neo4j)

It’s also included in the official [opens in new tabMCP Servers repository](https://github.com/modelcontextprotocol/servers?tab=readme-ov-file#%EF%B8%8F-official-integrations).

## An Interactive Example

Here’s a simple example of how you can use the MCP to query a Neo4j database from a LLM.

Imagine you have a Neo4j database with an IMDB-like set of movies, actors, directors, genres and movie ratings.

I configure the publicly accessible database in my Claude.ai desktop configuration and restart the client.

![](https://cdn-images-1.medium.com/max/1024/1*_uCb-D3O0Fa5CfdCZybrAA.png)

Now I want to find out which movies were directed by Quentin Tarantino, so I’ll just ask Claude about it.

> _Check my movie database for movies directed by Quentin Tarantino._

Claude will pop up a permission dialog that asks me if I want to allow the query to the Neo4j database (once or for the whole chat), and if I accept, it will run the query and return the results in a nice format.

![](https://cdn-images-1.medium.com/max/1024/1*nZ_94geL38hPyf8ODH0lig.png)

Then it answers:

![](https://cdn-images-1.medium.com/max/1024/1*GIHD854jB_Lo3NnION7eLA.png)

Claude answer for: _Check my movie database for movies directed by Quentin Tarantino._

_I’ll help you query the database for movies directed by Quentin Tarantino._

_View Result from read-neo4j-cypher from movies-neo4j (local)_

{  
  `query`: `  
MATCH (d:Person {name: 'Quentin Tarantino'})-[:DIRECTED]->(m:Movie)  
RETURN m.title, m.year  
ORDER BY m.year  
`  
}  
  
[{'m.title': 'Reservoir Dogs', 'm.year': 1992}, {'m.title': 'Pulp Fiction', 'm.year': 1994}, {'m.title': 'Four Rooms', 'm.year': 1995}, {'m.title': 'Jackie Brown', 'm.year': 1997}, {'m.title': 'Kill Bill: Vol. 1', 'm.year': 2003}, {'m.title': 'Kill Bill: Vol. 2', 'm.year': 2004}, {'m.title': 'Sin City', 'm.year': 2005}, {'m.title': 'Grindhouse', 'm.year': 2007}, {'m.title': 'Death Proof', 'm.year': 2007}, {'m.title': 'Inglourious Basterds', 'm.year': 2009}, {'m.title': 'Django Unchained', 'm.year': 2012}, {'m.title': 'The Hateful Eight', 'm.year': 2015}]

_Here are Quentin Tarantino’s directed films in chronological order:_

1. _Reservoir Dogs (1992)_
2. _Pulp Fiction (1994)_
3. _Four Rooms (1995)_
4. _Jackie Brown (1997)_
5. _Kill Bill: Vol. 1 (2003)_
6. _Kill Bill: Vol. 2 (2004)_
7. _Sin City (2005) …​_

_Let me know if you’d like to know more details about any of these films or if you’d like to explore other aspects of Tarantino’s work!_

> _What are other popular movies that the actors of Kill Bill acted in?_

![](https://cdn-images-1.medium.com/max/1024/1*P-QK_c6ayzraCmPsKK-IrQ.png)

Claude answer for: _What are other popular movies that the actors of Kill Bill acted in?_

_I’ll query for other notable movies featuring actors from Kill Bill: Vol. 1 and Kill Bill: Vol. 2._

_View Result from read-neo4j-cypher from movies-neo4j (local)_

{  
  `query`: `  
MATCH (kb:Movie)  
WHERE kb.title IN ['Kill Bill: Vol. 1', 'Kill Bill: Vol. 2']  
MATCH (actor:Person)-[:ACTED_IN]->(kb)  
MATCH (actor)-[:ACTED_IN]->(other:Movie)  
WHERE other.title <> 'Kill Bill: Vol. 1'  
AND other.title <> 'Kill Bill: Vol. 2'  
RETURN DISTINCT actor.name, collect(DISTINCT other.title) as other_movies  
ORDER BY actor.name  
`  
}  
[{'actor.name': 'Daryl Hannah', 'other_movies': ['Clan of the Cave Bear, The', 'Tie That Binds, The', 'Addams Family Reunion', 'Summer Lovers', 'Legal Eagles', 'Reckless', 'Pope of Greenwich Village, The', 'Splash', 'Roxanne', 'Gingerbread Man, The', 'My Favorite Martian', 'Steel Magnolias', 'Walk to Remember, A']}, {'actor.name': 'Lucy Liu', 'other_movies': ['Afro Samurai: Resurrection', 'Lucky Number Slevin', 'Mulan II', 'Cypher', "Charlie's Angels: Full Throttle", 'Ballistic: Ecks vs. Sever', "Charlie's Angels", 'Shanghai Noon']}, {'actor.name': 'Michael Parks', 'other_movies': ['Tusk', 'Death Wish 5: The Face of Death']}, {'actor.name': 'Uma Thurman', 'other_movies': ['Misérables, Les', 'My Super Ex-Girlfriend', 'Even Cowgirls Get the Blues', 'Truth About Cats & Dogs, The', 'Batman & Robin', 'Gattaca', 'Avengers, The', 'Month by the Lake, A', 'Jennifer 8', 'Mad Dog and Glory', 'Johnny Be Good', 'Henry & June', 'Be Cool', 'Producers, The', 'Tape', 'Paycheck']}, {'actor.name': 'Vivica A. Fox', 'other_movies': ['Set It Off', 'Soul Food', 'Boat Trip', 'Juwanna Mann', 'Why Do Fools Fall In Love?']}]

_Let me break down some notable movies by the main cast members:_

_Uma Thurman’s other major films include: — The Avengers — Gattaca — Batman & Robin — Les Misérables — The Producers — Be Cool_

_Lucy Liu appeared in: — Charlie’s Angels and Charlie’s Angels: Full Throttle — Shanghai Noon — Lucky Number Slevin — Mulan II_

_Michael Parks appeared in: — Tusk — Death Wish 5: The Face of Death_

_Would you like to know more details about any of these actors or specific movies? I can query for additional information like release years, directors, or other cast members._

> _Tell me more about Tusk; is it worth watching? Use the database schema to figure out the ratings._

![](https://cdn-images-1.medium.com/max/1024/1*B6D02g0WvSfUVsn7yV3yBA.png)

Claude answer for: _Tell me more about Tusk; is it worth watching? Use the database schema to figure out the ratings._

Now, it uses the other tool (get-neo4j-schema) available in the integration to get the database schema, which is important for determining where and how ratings are stored. It returns node types, their attributes, if they’re indexed, and the relationships to other nodes.

_Let me first check the database schema to see what kind of rating information is available about movies._

_View Result from get-neo4j-schema from movies-neo4j (local)_

{}  
[{'label': 'Movie', 'attributes': {'budget': 'INTEGER', 'movieId': 'STRING unique indexed', 'tmdbId': 'STRING unique indexed', 'plotEmbedding': 'LIST', 'imdbVotes': 'INTEGER', 'runtime': 'INTEGER', 'posterEmbedding': 'LIST', 'countries': 'LIST', 'imdbId': 'STRING indexed', 'url': 'STRING', 'plot': 'STRING indexed', 'released': 'STRING indexed', 'languages': 'LIST', 'imdbRating': 'FLOAT indexed', 'title': 'STRING indexed', 'poster': 'STRING', 'year': 'INTEGER indexed', 'revenue': 'INTEGER'}, 'relationships': {'IN_GENRE': 'Genre'}}, {'label': 'Genre', 'attributes': {'name': 'STRING unique indexed'}, 'relationships': {}}, {'label': 'User', 'attributes': {'name': 'STRING indexed', 'userId': 'STRING unique indexed'}, 'relationships': {'RATED': 'Movie'}}, {'label': 'Actor', 'attributes': {'born': 'DATE', 'bornIn': 'STRING', 'tmdbId': 'STRING', 'bio': 'STRING', 'died': 'DATE', 'name': 'STRING', 'poster': 'STRING', 'imdbId': 'STRING', 'url': 'STRING'}, 'relationships': {'ACTED_IN': 'Movie', 'DIRECTED': 'Movie'}}, {'label': 'Director', 'attributes': {'born': 'DATE', 'bornIn': 'STRING', 'tmdbId': 'STRING', 'bio': 'STRING', 'died': 'DATE', 'name': 'STRING', 'poster': 'STRING', 'imdbId': 'STRING', 'url': 'STRING'}, 'relationships': {'ACTED_IN': 'Movie', 'DIRECTED': 'Movie'}}, {'label': 'Person', 'attributes': {'born': 'DATE', 'bornIn': 'STRING', 'tmdbId': 'STRING unique indexed', 'bio': 'STRING indexed', 'died': 'DATE', 'name': 'STRING indexed', 'poster': 'STRING', 'imdbId': 'STRING', 'url': 'STRING'}, 'relationships': {'ACTED_IN': 'Movie', 'DIRECTED': 'Movie'}}]

_Now, let me query specific information about Tusk, including its rating, plot, and genre._

_View Result from read-neo4j-cypher from movies-neo4j (local)_

{  
  `query`: `  
MATCH (m:Movie {title: 'Tusk'})  
MATCH (m)-[:IN_GENRE]->(g:Genre)  
MATCH (d:Person)-[:DIRECTED]->(m)  
MATCH (a:Person)-[:ACTED_IN]->(m)  
RETURN m.title, m.year, m.imdbRating, m.plot, collect(DISTINCT g.name) as genres,  
       collect(DISTINCT d.name) as directors, collect(DISTINCT a.name) as actors  
`  
}  
[{'m.title': 'Tusk', 'm.year': 2014, 'm.imdbRating': 5.4, 'm.plot': 'When podcaster Wallace Bryton goes missing in the backwoods of Manitoba while interviewing a mysterious seafarer named Howard Howe, his best friend Teddy and girlfriend Allison team with an ex-cop to look for him.', 'genres': ['Comedy', 'Drama', 'Horror'], 'directors': ['Kevin Smith'], 'actors': ['Michael Parks', 'Haley Joel Osment', 'Genesis Rodriguez', 'Justin Long']}]

_Based on the database information, here’s what I can tell you about Tusk:_

_1. It’s a horror-comedy-drama directed by Kevin Smith (known for Clerks, Mallrats, Chasing Amy) and was released in September 2014._

_2. The plot follows a podcaster named Wallace Bryton who disappears in Manitoba while interviewing a mysterious seafarer named Howard Howe. His girlfriend and best friend then team up with an ex-cop to search for him._

_3. The movie has a relatively modest IMDb rating of 5.4/10, suggesting mixed to negative reception._

….

_The low IMDb rating suggests it’s not for everyone, but it has developed something of a cult following among fans of unusual horror films._

_Would you like me to find similar movies in the database that you might enjoy instead?_

> _Hmm, OK, give me the best movie of the list that we got from the Kill Bill actors with some detail._

![](https://cdn-images-1.medium.com/max/1024/1*-42ylI4L8kOLzpWyvjglSw.png)

Claude answer for: _Hmm, ok, give me the best movie of the list that we got from the Kill Bill actors with some detail._

_Let me search through those movies and their ratings to find the highest rated one._

{  
  `query`: `  
MATCH (a:Person)-[:ACTED_IN]->(kb:Movie)  
WHERE kb.title IN ['Kill Bill: Vol. 1', 'Kill Bill: Vol. 2']  
MATCH (a)-[:ACTED_IN]->(m:Movie)  
WHERE m.title <> 'Kill Bill: Vol. 1' AND m.title <> 'Kill Bill: Vol. 2'  
MATCH (m)-[:IN_GENRE]->(g:Genre)  
MATCH (d:Person)-[:DIRECTED]->(m)  
RETURN DISTINCT m.title, m.year, m.imdbRating, m.plot,  
       collect(DISTINCT g.name) as genres,  
       collect(DISTINCT d.name) as directors,  
       collect(DISTINCT a.name) as actors  
ORDER BY m.imdbRating DESC  
LIMIT 1  
`  
}  
[{'m.title': 'Gattaca', 'm.year': 1997, 'm.imdbRating': 7.8, 'm.plot': 'A genetically inferior man assumes the identity of a superior one in order to pursue his lifelong dream of space travel.', 'genres': ['Thriller', 'Sci-Fi', 'Drama'], 'directors': ['Andrew Niccol'], 'actors': ['Uma Thurman']}]

_From the movies we found earlier, Gattaca (1997) stands out as the highest-rated film with an IMDb rating of 7.8/10. Let me get some additional details about this film._

{  
  `query`: `  
MATCH (m:Movie {title: 'Gattaca'})  
MATCH (p:Person)-[:ACTED_IN]->(m)  
RETURN m.runtime, collect(p.name) as cast  
`  
}  
[{'m.runtime': 106, 'cast': ['Xander Berkeley', 'Gore Vidal', 'Uma Thurman', 'Ethan Hawke']}]

_Here’s what makes Gattaca stand out:_

_1. Plot: It’s a science fiction drama about a man born genetically “inferior” who assumes the identity of a genetically superior person to pursue his dream of space travel. The story explores themes of genetic discrimination, determination, and human potential._

…..

_Would you like me to find similar movies in the database that you might enjoy if you like Gattaca?_

> _What is the other average, min, max user rating for Gattaca?_

![](https://cdn-images-1.medium.com/max/1024/1*tMPo-q3SQ_HL4UffjPs6RA.png)

Claude answer for: _What is the other average, min, max user rating for Gattaca?_

_I’ll look up the user ratings statistics for Gattaca._

{  
  `query`: `  
MATCH (m:Movie {title: 'Gattaca'})  
MATCH (u:User)-[r:RATED]->(m)  
RETURN  
  m.title,  
  count(r) as number_of_ratings,  
  round(avg(r.rating)*10)/10 as avg_rating,  
  min(r.rating) as min_rating,  
  max(r.rating) as max_rating  
`  
}  
[{'m.title': 'Gattaca', 'number_of_ratings': 84, 'avg_rating': 3.7, 'min_rating': 1.0, 'max_rating': 5.0}]

_I’ll look up the user ratings statistics for Gattaca._

_Here are the user rating statistics for Gattaca:  
– Average User Rating: 3.68 out of 5  
– Minimum Rating: 1.0  
– Maximum Rating: 5.0  
– Number of Ratings: 84 users_

_This average rating of 3.68/5 (equivalent to about 7.4/10) aligns fairly well with the IMDb rating of 7.8/10 we saw earlier. The movie has received the full range of possible scores (1–5), showing that while most people rate it highly, there are some varying opinions. With 84 ratings in the database, this gives us a decent sample size to work with._

> _Ok I watched Gattaca and loved it, what other movies are similar to it? Can you create a user for me with an ID mesirii and store my rating of 8 out of 10 (normalize it to fit the other ratings for the movie)?_

We’re up for write statements, and we get another permission check for the write operation tool. Here is the first time Claude makes a small syntax mistake in its Cypher statement and immediately fixes it. As my database is read only, it cannot actually write to it, but it offers me other movies that I can watch next.

![](https://cdn-images-1.medium.com/max/1024/1*cRTf53ROUR4SFuHHkMwpBQ.png)

_I’ll help you add your rating first, normalizing 8/10 to 4/5 for consistency with the database’s rating scale._

_I apologize for the error with creating a user rating — it seems I don’t have permission to create new users in the database. However, I can still help you find similar movies to Gattaca!_

_View Result from write-neo4j-cypher from movies-neo4j (local)_

{  
  `query`: `  
MERGE (u:User {userId: 'mesirii', name: 'mesirii'})  
MERGE (m:Movie {title: 'Gattaca'})  
CREATE (u)-[:RATED {rating: 4.0}]->(m)  
RETURN u.name, m.title  
`  
}

_I apologize for the error — it seems I don’t have permission to create new user records. Let me focus instead on finding similar movies to Gattaca. I’ll search for movies that share similar genres, themes, and ratings:_

_View Result from read-neo4j-cypher from movies-neo4j (local)_

{  
  `query`: `  
MATCH (m:Movie)-[:IN_GENRE]->(g:Genre)  
WHERE g.name IN ['Sci-Fi', 'Drama']  
AND m.title <> 'Gattaca'  
AND m.imdbRating >= 7.0  
WITH m, collect(DISTINCT g.name) as genres  
WHERE size(genres) >= 2  
RETURN m.title, m.year, m.imdbRating, m.plot, genres  
ORDER BY m.imdbRating DESC  
LIMIT 5  
`  
}  
[{'m.title': 'Inception', 'm.year': 2010, 'm.imdbRating': 8.8, 'm.plot': 'A thief, who steals corporate secrets through use of dream-sharing technology, is given the inverse task of planting an idea into the mind of a CEO.', 'genres': ['Drama', 'Sci-Fi']}, {'m.title': 'Black Mirror', 'm.year': None, 'm.imdbRating': 8.8, 'm.plot': 'A television anthology series that shows the dark side of life and technology.', 'genres': ['Drama', 'Sci-Fi']}, {'m.title': "Dr. Horrible's Sing-Along Blog", 'm.year': 2008, 'm.imdbRating': 8.7, 'm.plot': 'An aspiring supervillain must balance his career and his pursuit of a beautiful do-gooder.', 'genres': ['Drama', 'Sci-Fi']}, {'m.title': 'Battlestar Galactica', 'm.year': 2003, 'm.imdbRating': 8.7, 'm.plot': 'A re-imagining of the original series in which a rag-tag fugitive fleet of the last remnants of mankind flees pursuing robots while simultaneously searching for their true home, Earth.', 'genres': ['Drama', 'Sci-Fi']}, {'m.title': 'Prestige, The', 'm.year': 2006, 'm.imdbRating': 8.5, 'm.plot': 'Two stage magicians engage in competitive one-upmanship in an attempt to create the ultimate stage illusion.', 'genres': ['Sci-Fi', 'Drama']}]

_Let me search for movies with similar themes, genres, and ratings:_

{  
  `query`: `  
MATCH (m:Movie {title: 'Gattaca'})  
MATCH (m)-[:IN_GENRE]->(g:Genre)  
MATCH (other:Movie)-[:IN_GENRE]->(g)  
WHERE other.title <> 'Gattaca'   
  AND other.imdbRating >= 7.0  
  AND exists(other.plot)  
RETURN DISTINCT other.title, other.imdbRating, other.plot,   
  collect(DISTINCT g.name) as genres  
ORDER BY other.imdbRating DESC  
LIMIT 5  
`  
}  
  
Error: {code: Neo.ClientError.Statement.SyntaxError} {message: The property existence syntax `... exists(variable.property)` is no longer supported. Please use `variable.property IS NOT NULL` instead. (line 7, column 7 (offset: 174))  
"  AND exists(other.plot)"  
       ^}

_I apologize for the syntax error. Let me correct the query:_

_Let me refine this search to focus more specifically on sci-fi and thriller elements like Gattaca:_

{  
  `query`: `  
MATCH (m:Movie {title: 'Gattaca'})  
MATCH (m)-[:IN_GENRE]->(g:Genre)  
MATCH (other:Movie)-[:IN_GENRE]->(g)  
WHERE other.title <> 'Gattaca'   
  AND other.imdbRating >= 7.0  
  AND other.plot IS NOT NULL  
RETURN DISTINCT other.title, other.imdbRating, other.plot,   
  collect(DISTINCT g.name) as genres  
ORDER BY other.imdbRating DESC  
LIMIT 5  
`  
}  

_Based on your enjoyment of Gattaca, here are some highly-rated recommendations that share similar themes or elements:_

_1. Black Mirror (8.8/10)  
 — A thought-provoking sci-fi anthology series that, like Gattaca, explores the dark implications of technology on society  
 — Each episode tells a different story about how technology might affect human society_

….

_These films all share Gattaca’s thoughtful approach to science fiction, focusing more on human drama and ethical questions rather than action. They all explore themes of identity, humanity, and the impact of technology/science on society, which were central to Gattaca._

_Would you like more details about any of these recommendations?_

…

That’s my first experience with the MCP and the Neo4j connector. Pretty cool, huh? It was a great experience, and I can see how this can be used to enhance the conversational experience with LLMs like Claude.

## The Model Context Protocol

How does this work under the hood?

There is comprehensive documentation of the MCP on the modelcontextprotocol.io site, but here’s a brief overview:

![](https://cdn-images-1.medium.com/max/1024/1*fbiVQnaxHWBKRpdnoXJPVw.png)

Architecture diagram from the MCP documentation

MCP follows a [opens in new tabclient-server architecture](https://modelcontextprotocol.io/docs/concepts/architecture) where:

- **Hosts** are LLM applications (like Claude Desktop or IDEs) that initiate connections and use the client.
- **Clients** maintain 1:1 connections using the transport protocol with servers from within the host application.
- **Servers** provide context, tools, and prompts to clients.

The protocol layer handles message framing, request/response linking, notifications and high-level communication patterns.

The MCP allows for different transport protocols. Those currently supported are HTTPS (with Server-Sent-Events (SSE) for server to client messages and HTTP POST for client to server) and STDIO (standard-in/out) for local servers where the server is started by the client and can communicate via stdin/stdout.

All transport message exchanges are based on a [opens in new tabspecification](https://spec.modelcontextprotocol.io/specification/) using JSON-RPC 2.0, so it encourages us to implement the protocol in other languages or transport layers.

The based message types are:

- **Resources**: Context and data for the user or the AI model to use
- **Prompts**: Templated messages and workflows for users
- **Tools**: Functions for the AI model to execute
- **Sampling**: Server-initiated agentic behaviors and recursive LLM interactions

Additional relevant aspects include configuration, progress tracking, cancellation, error reporting, and logging.

The protocol spec is also considering security and trust, which is important when allowing LLMs’ access to external data sources. With write access to databases and file systems and servers running locally, and the potential for malicious code execution, security should be high priority. The foundation models are known to be vulnerable to adversarial attacks, prompt injections, and hallucinations.

LLM users are often non-technical and might not be aware of the risks involved in allowing an AI model to access their data.

That’s why the protocol employs the following key principles:

- User consent and control: Users need to approve server and data access and always be able to discern which data was sent and received. Client implementations need to have clear UIs for these authorizations.
- Data privacy access control for user data, no data exfiltration to other services by hosts and servers.
- Tool safety is used to prevent malicious code execution and to ensure that tools are safe to run and don’t have side effects. Tools need to be structured in a way that a user clearly understands what each tool is doing.
- LLM sampling controls: The protocol limits server access to user prompts. Users can control sampling and what data and prompts are sent to the server.

## Neo4j MCP Server Implementation

When we implemented the Neo4j MCP server, we had to implement three components:

1. Connection to the Neo4j database and executing Cypher queries
2. Registering and listing the available tools
3. Handling the tool execution requests

### Connection and Cypher Execution With Neo4j Database

In our case, we wrote a small neo4jDatabase class that holds the Neo4j Python driver to connect to the database, execute the statements, and render the results as JSON text block.

def _execute_query(self, query: str, params: dict[str, Any] | None = None) -> list[dict[str, Any]]:  
    """Execute a Cypher query and return results as a list of dictionaries"""  
    logger.debug(f"Executing query: {query}")  
    try:  
        result = self.driver.execute_query(query, params)  
        if is_write_query(query):  
            counters = vars(result.summary.counters)  
            logger.debug(f"Write query affected {counters}")  
            return [counters]  
        else:  
            results = [dict(r) for r in result.records]  
            logger.debug(f"Read query returned {len(results)} rows")  
            return results  
    except Exception as e:  
        logger.error(f"Database error executing query: {e}n{query}")  
        raise

### Register Available Tools for Interacting With the Database

These tools will later be available to the LLM to choose from:

- get-neo4j-schema
- read-neo4j-cypher
- write-neo4j-cypher

Each has a description and possible parameters (in our case, just a mandatory “query” parameter for the Cypher tools). We can imagine a number of additional generic tools for a graph database like getting the aggregated neighborhood of a node, finding shortest paths, or running graph algorithms.

Here’s the definition of the write-neo4j-cypher tool:

@server.list_tools()  
    async def handle_list_tools() -> list[types.Tool]:  
        """List available tools"""  
        return [  
            types.Tool(  
                name="write-neo4j-cypher",  
                description="Execute a write Cypher query on the neo4j database",  
                inputSchema={  
                    "type": "object",  
                    "properties": {  
                        "query": {"type": "string", "description": "Cypher write query to execute"},  
                    },  
                    "required": ["query"],  
                },  
            ),  
#            ...  
        ]

### Execution of the Registered Tools

When a user asks Claude to query the database, Claude asks the user for permission, starts the server if it’s not already running as a local server, and sends a request to the MCP server with the tool name and parameters via the transport protocol.

In the tool implementations, we just use this class to execute the statements. There is a small check that we only allow read statements in the read tool and vice versa (the write tool only returns update statistics).

@server.call_tool()  
    async def handle_call_tool(  
        name: str, arguments: dict[str, Any] | None  
    ) -> list[types.TextContent | types.ImageContent | types.EmbeddedResource]:  
        """Handle tool execution requests"""  
        try:  
            if name == "get-neo4j-schema":  
                results = db._execute_query(  
                    """  
CALL apoc.meta.data() yield label, property, type, other, unique, index, elementType  
WHERE elementType = 'node'  
RETURN label,  
    collect(case when type <> 'RELATIONSHIP' then [property, type] end) as attributes,  
    collect(case when type = 'RELATIONSHIP' then [property, head(other)] end) as relationships  
                    """  
                )  
                return [types.TextContent(type="text", text=str(results))]

So you can see it’s pretty straightforward to implement an MCP server for a tool of your choice.

## From Conversation to Code

Combining MCP tool results with Claude’s code generation and execution capabilities is taking the interaction to the next level.

One thing that’s already impressive in the interactions is that the previous tool call data results are part of the conversational history, so the LLM makes use of them to shape queries and input parameters for further server interactions.

But it goes even beyond this with the code-generation capabilities, which allow you to easily render results as charts, networks, or just compute derived data to export. It generates artifacts of JavaScript using React or other libraries combined with charting libraries that you can render and visualize directly.

Here are two examples of generating a heat map per movie or per rating:

![](https://cdn-images-1.medium.com/max/1024/1*LItlnYmWNjC3KGYT3E2Elw.png)

Heat-map rendering of Tarantino movies

> _Actually I wanted ratings and years as a square heatmap._

![](https://cdn-images-1.medium.com/max/1024/1*6a1W5KdhOK3dcw69n_vrNA.png)

Square heat map of rating-count per rating-value

And finally a network visualization of our conversation:

> _Generate a graph visualization of the conversation so far, starting with Quentin Tarantino’s movies to the recommendations all the way to the final suggestions. Please make sure the rendering has high contrast._

![](https://cdn-images-1.medium.com/max/1024/1*6WQO0D2OxLgapo9UYhv3ZA.png)

## Summary

We’d love for you to test out the Neo4j MCP server implementation and share your experiences, give us [opens in new tabfeedback on GitHub](https://github.com/neo4j-contrib/mcp-neo4j), and perhaps write an article of your own.

Happy experimenting!

---

[opens in new tabClaude Conversing With Neo4j Via MCP](https://medium.com/neo4j/claude-conversing-with-neo4j-via-mcp-5419d24bb7a0) was originally published in [opens in new tabNeo4j Developer Blog](https://medium.com/neo4j) on Medium, where people are continuing the conversation by highlighting and responding to this story.