```python


    # NOTE:
    # In the original production setup this sync logic was implemented
    # as a PostgreSQL stored procedure.
    # For this demo/portfolio version, the procedure has been inlined
    # as a single UPSERT query (INSERT ... ON CONFLICT) so that
    # reviewers can see the ETL logic directly in Python + SQL.



import requests
from datetime import datetime

from db import get_connection 
from config import SHOPIFY_STORE, SHOPIFY_API_VERSION, HEADERS  

def get_sku_map():
    """
    Fetch all product variants from Shopify and build a mapping:
    inventory_item_id -> SKU.
    """
    print(" Fetching all SKU mapping information...")
    all_variants = []
    next_url = f"https://{SHOPIFY_STORE}/admin/api/{SHOPIFY_API_VERSION}/variants.json?limit=250"

    while next_url:
        response = requests.get(next_url, headers=HEADERS)
        response.raise_for_status()
        data = response.json()
        all_variants.extend(data["variants"])

        link_header = response.headers.get("Link")
        if link_header and 'rel="next"' in link_header:
            parts = link_header.split(",")
            next_link = [p for p in parts if 'rel="next"' in p]
            next_url = next_link[0].split(";")[0].strip().strip("<>") if next_link else None
        else:
            next_url = None

    print(f" Received {len(all_variants)} variants (SKUs) in total")
    return {v["inventory_item_id"]: v["sku"] for v in all_variants}


def save_inventory_to_db(records):
    """
    Persist flattened inventory records into PostgreSQL.

    Each record contains:
      - inventory_item_id
      - sku
      - location_id
      - location_name
      - available (quantity)
    """
    print(" Starting DB save...")

    conn = get_connection()
    cur = conn.cursor()


    cur.execute("TRUNCATE TABLE shopify_inventory CASCADE;")

    for item in records:
        # print(f"→ Saving: {item['sku']} @ {item['location_name']} ({item['available']} units)")
        cur.execute(
            """
            INSERT INTO shopify_inventory (
                inventory_item_id,
                sku,
                location_id,
                location_name,
                available_quantity
            )
            VALUES (%s, %s, %s, %s, %s)
            ON CONFLICT (inventory_item_id, location_id)
            DO UPDATE SET
                sku = EXCLUDED.sku,
                location_name = EXCLUDED.location_name,
                available_quantity = EXCLUDED.available_quantity
            """,
            (
                item["inventory_item_id"],
                item["sku"],
                item["location_id"],
                item["location_name"],
                item["available"],
            ),
        )

    conn.commit()
    cur.close()
    conn.close()
    print(f"✅ Finished saving {len(records)} inventory records to DB")


def sync_inventory():
    """
    End-to-end inventory sync:
    1) Load all locations from Shopify
    2) Load inventory levels per location
    3) Join with SKU mapping (inventory_item_id -> SKU)
    4) Upsert into PostgreSQL
    """
    try:
        start_time = datetime.now()
        print(f"🚀 Inventory sync started: {start_time}")

        locations = get_locations()      # your own helper
        sku_map = get_sku_map()
        all_inventory = []

        for loc in locations:
            loc_id = loc["id"]
            loc_name = loc["name"]
            inventory = get_inventory_levels(loc_id)  # your own helper

            for item in inventory:
                inv_id = item["inventory_item_id"]
                sku = sku_map.get(inv_id, "UNKNOWN")
                available = item["available"]

                all_inventory.append(
                    {
                        "inventory_item_id": inv_id,
                        "sku": sku,
                        "available": available,
                        "location_id": loc_id,
                        "location_name": loc_name,
                    }
                )

        if all_inventory:
            save_inventory_to_db(all_inventory)
        else:
            print(" No inventory data fetched.")

        end_time = datetime.now()
        print(f" Inventory sync completed: {end_time}")
        print(f" Total duration: {end_time - start_time}")
    except Exception as e:
        print("❌ Error during inventory sync:", e)
       
        raise e


```
