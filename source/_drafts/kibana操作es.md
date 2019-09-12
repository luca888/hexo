kibana操作es
---

index: gis-marker
type: docs

查询:
GET /gis-marker/_search

删除:
POST gis-marker/docs/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}