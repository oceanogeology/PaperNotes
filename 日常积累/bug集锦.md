# BUG集锦

*bug表现：pytorch多卡训练，0卡爆满，OOM。*

解决办法：state_dict = torch.load(args.pretrained_model1, map_location="cpu")

------

