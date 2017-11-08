# чпу wishlist
controler/footer.php

            $i = 0;

            foreach($data['wishlist'] as &$item)
            {
                $id = $item['product_id'];
                // получаем алиас товара
                $product_alias=$this->model_catalog_product->getSeoUrlProduct($id);
                $data['path'][$i] = array();
                // пушим
                array_push($data['path'][$i], $product_alias);
                // получаем категорию товара
                $category =$this->model_catalog_product->getCategories($id);
                $category_alias=$this->model_catalog_product->getSeoUrlCategory($category[0]['category_id']);
                array_push($data['path'][$i], $category_alias);
                // строим дерево категорий
                $parent =$this->model_catalog_product->buildTree($category[0]['category_id']);
                array_push($data['path'][$i], $parent);
                $data['tree'][$i] = array_reverse($data['path'][$i]);
                $data['wishlist_tree'] = implode("/",$data['tree'][$i]);
                $item['path'] = 'http://' . $this->request->server['HTTP_HOST'] .'/'.$data['wishlist_tree'];
                $i++;
            }
			
			
			
model/product.php


    public function getParentCategories($category_id) {
        $query = $this->db->query("SELECT parent_id FROM " . DB_PREFIX . "category WHERE " . DB_PREFIX . "category.category_id = '" . (int)$category_id . "'");
        if(!empty($query->row['parent_id']))
        {
            $a = $this->getParentCategories($query->row['parent_id']);
            $seo = $this->getSeoUrlCategory($query->row['parent_id']);
        }
        return $query->row['parent_id'];
	}
    public function buildTree($category_id)
    {
        $tree = array();
        $parent = $this->getParentCategories($category_id);
        $seo = $this->getSeoUrlCategory($parent);
        array_push($tree, $seo);
        while($parent!=0)
        {
            $parent = $this->getParentCategories($parent);
            if($parent>0)
            {
                $seo = $this->getSeoUrlCategory($parent);
                        array_push($tree, $seo);
            }

        }
        $path = implode("/",$tree);
        return $path;
    }
    public function getSeoUrlCategory($category_id) {
            $query = $this->db->query("SELECT keyword FROM " . DB_PREFIX . "url_alias WHERE query = 'category_id=" . (int)$category_id . "'");
            return $query->row['keyword'];
	}
    public function getSeoUrlProduct($product_id) {
            $query = $this->db->query("SELECT keyword FROM " . DB_PREFIX . "url_alias WHERE query = 'product_id=" . (int)$product_id . "'");
            return $query->row['keyword'];
	}