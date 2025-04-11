// nano ~/Library/Application\ Support/Claude/claude_desktop_config.json

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




