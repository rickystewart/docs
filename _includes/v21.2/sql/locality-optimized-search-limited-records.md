{% if page.name == "cost-based-optimizer.md" %} Locality-optimized search {% else %} [Locality-optimized search](cost-based-optimizer.html#locality-optimized-search-in-multi-region-clusters) {% endif %} only works for queries selecting a limited number of records (up to 100,000 unique keys). Also, It does not yet work with [`LIMIT`](limit-offset.html) clauses.

[Tracking GitHub Issue](https://github.com/cockroachdb/cockroach/issues/64862)
