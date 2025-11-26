```python


def update_variant_preorder_metafields(variant_gid: str, eta_date: str, incoming_qty: int):
    """
    Update Shopify product variant metafields:
    - custom.eta (date)
    - custom.incoming_qty (integer)
    """
    mutation = """
    mutation setPreorder($id: ID!, $eta: Date!, $qty: Int!) {
      productVariantUpdate(input: {
        id: $id,
        metafields: [
          { namespace: "custom", key: "eta", value: $eta, type: "date" },
          { namespace: "custom", key: "incoming_qty", value: $qty, type: "number_integer" }
        ]
      }) {
        productVariant { id }
        userErrors { field message }
      }
    }
    """
    variables = {"id": variant_gid, "eta": eta_date, "qty": incoming_qty}
    resp = shopify_graphql(mutation, variables)
    return resp

```
