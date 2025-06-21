`add_index :events, :owner`
- 既存の owner カラムにインデックスを追加するマイグレーションの書き方。
- 検索やJOINの高速化のためにインデックスを付けます。
- 外部キー制約は付きません。

> Add an index to the owner_id column for faster lookups  
> This is useful for queries that filter events by their owner  
> Adding an index on owner_id can significantly improve query performance  
> when searching for events by owner, especially if the table grows large  
> This index will allow the database to quickly locate all events associated with a specific owner  
> This is particularly important for applications with many users and events  
> It helps in optimizing queries that involve filtering events by their owner  
> The index will be created on the owner_id column, which is a foreign key reference to the users table  
> This will speed up queries that filter events by owner_id  
> It is a good practice to add indexes on foreign key columns to improve query performance