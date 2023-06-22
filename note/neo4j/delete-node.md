> NOTE: Nodes cannot be deleted if they have relationships, so you need to detach the nodes to delete them.
1. Delete all Movie and Person nodes, and their relationships.
`MATCH (n) DETACH DELETE n`
2. Verify that the Movie Graph has been removed.
`MATCH (n) RETURN n`
