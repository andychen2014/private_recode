# private_recode
公司组织枚举算法

 //0表示隐藏
        //1表示显示
        public void InitOrg(int status)
        {
            String sql = "";
            MySqlCommand cmd = new MySqlCommand();
            DataTable dt = new DataTable();
            if (Settings.Default.app_user_isSuper == 1)
            {
                sql = "SELECT org_id,org_name,parent_id FROM lockset.organization where 1 = 1  ";
                if (status == -1)
                {
                    //无条件显示
                }
                else
                {
                    //默认是显示没有隐藏的。
                    sql += " and status =@status";
                    cmd.Parameters.Add(new MySqlParameter("@status", status));
                }
                cmd.CommandText = sql;
                dt = MySqlHelper.ExecuteDataTable(MySqlHelper.getConnectionStringManager(), cmd, null);
            }
            else
            {
                sql = "SELECT org_id,org_name,parent_id FROM lockset.organization where 1= 1";
                if (status == -1)
                {
                    //无条件显示
                }
                else
                {
                    //默认是显示没有隐藏的。
                    sql += " and status =@status";
                    sql += " and org_id =@org_id or org_id = @branch_id or parent_id = @parent_id";
                    cmd.Parameters.Add(new MySqlParameter("@status", status));
                    
                }
                
                cmd.Parameters.Add(new MySqlParameter("@org_id",(object)Settings.Default.app_user_org_id));
                cmd.Parameters.Add(new MySqlParameter("@branch_id",(object)Settings.Default.app_user_branch_id));
                cmd.Parameters.Add(new MySqlParameter("@parent_id",(object)Settings.Default.app_user_branch_id));
                cmd.CommandText = sql;
                dt = MySqlHelper.ExecuteDataTable(MySqlHelper.getConnectionStringManager(), cmd, null);
                
            }
             
            if (dt != null)
            {
                int org_id = 0;
                String org_name = "";
                int parent_id = 0;

                //可编辑状态
                treeView1.LabelEdit = true;
                treeView1.Nodes.Clear();
                //增加一个节点，这个结点是根节点。
                TreeNode root = new TreeNode();
                foreach (DataRow dr in dt.Rows)
                {
                    //先获取org_id,再根据parent_id判断有无交节点
                    Array array = dr.ItemArray;
                    if (Convert.ToInt32(array.GetValue(2)) == 0)  //找到根节点{
                    {
                        root.Text = array.GetValue(1).ToString();
                        root.Tag = Convert.ToInt32(array.GetValue(0));
                        treeView1.Nodes.Add(root);

                        org_id = Convert.ToInt32(array.GetValue(0));
                        org_name = Convert.ToString(array.GetValue(1));
                        parent_id = Convert.ToInt32(array.GetValue(2));
                    }
                }
                fun_CreateTree(dt, root, org_id);
            }

        }

        public void fun_CreateTree(DataTable dt, TreeNode node, int i_org_id)
        {
            try
            {
                Console.WriteLine("tree node");
                int org_id = 0;
                String org_name = "";
                int parent_id = 0;
                foreach (DataRow dr in dt.Rows)
                {
                    //先获取org_id,再根据parent_id判断有无交节点
                    Array array = dr.ItemArray;
                    if (Convert.ToInt32(array.GetValue(2)) == i_org_id)
                    {
                        //增加一个节点，这个结点是根节点
                        TreeNode node_1 = new TreeNode();
                        node_1.Text = array.GetValue(1).ToString();
                        node_1.Tag = Convert.ToInt32(array.GetValue(0));
                        node.Nodes.Add(node_1);
                        org_id = Convert.ToInt32(array.GetValue(0));
                        org_name = Convert.ToString(array.GetValue(1));
                        parent_id = Convert.ToInt32(array.GetValue(2));
                    }
                }
                if (node.FirstNode != null)
                {
                    fun_CreateTree(dt, node.FirstNode, Convert.ToInt32(node.FirstNode.Tag));
                }
                else if (node.NextNode != null)
                {
                    fun_CreateTree(dt, node.NextNode, Convert.ToInt32(node.NextNode.Tag));
                }
            }
            catch (Exception ex)
            {
 //               MessageBox.Show(ex.ToString());
            }
        }
